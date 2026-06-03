# PEL-010 — Native ssh.exe MCP Wrappers Fail Silently on Windows

**Date logged:** June 2, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop + SSH MCP)
**Related entries:** PEL-008 (Silent mcpServers Block Deletion) and PEL-009 (Corporate SSL Inspection Silently Breaks Node MCP Attach) cover earlier failure modes in the same SSH MCP setup arc. PEL-009 was about getting the MCP server to attach at all; PEL-010 is about what happens after attach succeeds — the MCP's tools appear in Claude's catalog, but every actual SSH call returns empty output and exit 255. PEL-011 covers Claude's own diagnostic blind spots that compounded the time-to-resolution on this arc.

---

## Symptom

After resolving the SSL inspection cert issue (PEL-009) by pre-installing `@aiondadotcom/mcp-ssh` globally, the SSH MCP server attaches cleanly: no popup, `tools/list` returns the expected `listKnownHosts`, `checkConnectivity`, `runRemoteCommand`, etc. The MCP log shows successful initialize handshake. `listKnownHosts` correctly reads `~/.ssh/config` and returns the expected host alias with the right user, IP, and identity file path.

Every actual SSH call — `runRemoteCommand`, `checkConnectivity` — returns:

```json
{
  "stdout": "",
  "stderr": "",
  "code": 255
}
```

Exit code 255 is SSH's standard "couldn't establish session" code. Empty stderr is the anomaly. SSH normally writes "Permission denied," "Connection refused," "Host key verification failed," or at minimum a verbose banner. Completely empty stderr almost never occurs from native ssh.exe in normal failure paths.

Critically, the same SSH invocation from interactive PowerShell on the same machine, at the same moment, works:

```
PS C:\Users\dgreg> ssh harmonydesk whoami
ubuntu
```

And the same host accepts the same key from Termius without issue. So infrastructure, credentials, key validity, security groups, and routing are all healthy. The failure is specifically in the MCP→native-ssh.exe path.

Adding `MCP_SILENT=false` to the MCP env (to surface Aionda's verbose logs) and `LogLevel VERBOSE` to `~/.ssh/config` (to surface ssh's own debug output) changes nothing — stderr remains empty across multiple retries.

## Root cause

Two distinct things compound:

**1. Native-ssh-spawn from a Claude-Desktop child process is fragile on Windows.** Aionda's mcp-ssh uses `child_process.spawn(..., { shell: true })` to invoke `ssh.exe`. On Windows, `shell: true` routes through `cmd.exe`. When Claude Desktop spawns this child, the env that child inherits is not equivalent to the env an interactive PowerShell session provides. Several axes diverge:

- `USERPROFILE` / `HOME`: ssh uses these to locate `~/.ssh/config` and `~/.ssh/known_hosts`. If they differ, ssh reads a different config or none at all.
- No tty: ssh writes interactive prompts (host key acceptance, password, passphrase) to `/dev/tty` rather than stderr. No tty → silent exit when any such prompt is needed.
- Windows ACL key permission checks: OpenSSH on Windows enforces strict ACLs on private key files. The checks may pass differently depending on the security context the child process runs in vs. an interactive logon.
- PATH: even though Claude Desktop's log confirms `C:\Windows\System32\OpenSSH\` is in the child's PATH, child-process PATH and interactive PATH are not guaranteed identical across spawn layers.

Any one of these can produce a silent exit 255. The user has no signal which one fired.

**2. The wrapper layer has a diagnostic gap in the failure path.** Aionda's tool response surfaces `stdout`/`stderr`/`code` in JSON, but in this failure mode all three are empty/255. We don't have proof of whether (a) ssh ran and produced no output before silently exiting, or (b) ssh's output was captured by the wrapper but discarded, or (c) ssh.exe was never spawned successfully and cmd.exe returned 255 from a layer before ssh got a chance to run. Aionda's `MCP_SILENT=false` mode controls Aionda's own logging, not its capture of stderr from the wrapped process. The data needed to distinguish these cases is not exposed by the wrapper, which makes the failure non-debuggable from outside the package.

The compounding effect: even if the user could identify and fix the env propagation cause, the wrapper's diagnostic gap prevents them from knowing what to fix. Standard debugging loops (read the error message, fix the cause, retry) fail because there is no error message to read.

## Diagnostic technique

Recognition order:

1. **Empty stderr with exit 255 from any wrapped-ssh tool is the smoking gun.** Native ssh.exe almost never fails silently in practice. If you see empty stderr from a tool that wraps ssh, the wrapper is somehow at fault — either it can't surface the wrapped process's output, or the wrapped process is hitting a silent-exit edge that wouldn't occur from an interactive shell.

2. **Verify infrastructure is healthy via a parallel SSH client.** Termius, PuTTY, or PowerShell's native `ssh` are all valid. If at least one reaches the host and runs commands, server-side hypotheses (instance state, security group, key validity, user, DNS) are all ruled out. The failure is local to the MCP path.

3. **Check the MCP server log for the spawned command line.** `%APPDATA%\Claude\logs\mcp-server-<name>.log` records the `spawn` arguments Claude Desktop used. Replicating the spawn verbatim from PowerShell — e.g., `cmd /c <args>` — should reproduce the failure if the root cause is env/PATH. If it works in PowerShell, the issue is something Claude Desktop's spawn does that interactive cmd doesn't replicate (security context, env stripping).

4. **Don't iterate on the wrapper's debug flags.** If stderr is empty in the first failure, additional verbosity flags on the wrapper rarely help — the wrapper's gap is in the failure path itself, not the verbosity path. Time spent here is wasted.

## Canonical fix

For SSH-over-MCP specifically on Windows: use **`tufantunc/ssh-mcp`** (npm package `ssh-mcp`) instead of `@aiondadotcom/mcp-ssh` or any other native-ssh-spawning wrapper.

tufantunc uses the pure-JavaScript `ssh2` library, which implements the SSH protocol entirely in Node. There is no `ssh.exe` subprocess, no `cmd.exe` shell wrapper, no env propagation surface area. The MCP server opens a TCP socket directly to port 22, performs the SSH handshake in JS, and runs commands via the in-process protocol implementation.

Config block (after `npm install -g ssh-mcp` per PEL-009):

```json
"ssh": {
  "command": "C:\\Users\\<user>\\AppData\\Roaming\\npm\\ssh-mcp.cmd",
  "args": [
    "--host=<ip>",
    "--port=22",
    "--user=<user>",
    "--key=C:\\Users\\<user>\\.ssh\\<key>.pem"
  ]
}
```

Trade-offs:

- **Lost:** auto-discovery of `~/.ssh/config` aliases. tufantunc takes explicit `--host`/`--port`/`--user`/`--key` args in the JSON config. For a single host this is fine. For many hosts, credentials and paths would need to be repeated per host in the MCP config — at that point a different solution (per-host config blocks or a multi-host MCP wrapper that uses ssh2) becomes preferable.
- **Gained:** zero Windows-native-ssh surface area. Pure-JS protocol stack is immune to USERPROFILE drift, ACL checks, tty allocation, PATH propagation, and stderr buffering. Every silent-failure edge of native ssh.exe disappears.

**Generalization beyond SSH:** for any MCP that wraps a Windows-native tool, prefer implementations that re-implement the protocol in pure JS (or pure Go, etc.) over those that shell out via `spawn({shell:true})`. Wrapper-of-native-tool patterns are reliable on macOS and Linux because the env/tty/permission models are simpler. On Windows the wrapper pattern produces a class of silent-failure edges that JS-native protocol stacks structurally avoid.

## Operational consequence

The Aionda failure mode is particularly costly because it **masquerades as a successful MCP setup**. The popup from PEL-009 told the user clearly "attach failed." This failure mode has no popup, no log error, no missing tools — just empty output from every actual call. The user reasonably believes the SSH MCP is operational and starts adjusting things that aren't broken (config syntax, ssh keys, security groups, AWS console) while the actual fault is structural to the wrapper. The debugging time cost is multiple-x what a clean attach-failure would be.

Secondary cost: the Aionda package has a CVE history that points at exactly this architectural choice. CVE-2025-9654 (command injection up to 1.0.3) and GHSA-p4h8-56qp-hpgv (Windows-specific local-RCE via `spawn({shell:true})` with insufficient argument sanitization, patched in 1.3.5) both stem from the "shell out to native tool with user-controllable args, on Windows" pattern. The CVEs are patched, but the architecture that produced them is unchanged. Pure-JS protocol implementations don't expose the same attack surface because there's no shell layer to abuse.

## Recurrence prevention

- **Standing rule for this laptop: SSH MCP is `ssh-mcp` (tufantunc), not `@aiondadotcom/mcp-ssh` (Aionda).** Do not retry Aionda even if a future version claims to fix the spawn issue. The structural choice to shell out to native ssh.exe is the problem, and patches won't change the architecture. Recorded in venture memory edit for cross-session persistence.

- **When comparing two MCP servers that wrap the same native tool on Windows, prefer the one that re-implements the protocol in pure JS over the one that shells out.** Look for `ssh2`, `node-ssh`, `node-pty`, or similar pure-JS protocol libraries in the package's `dependencies`. If the package's source shows `child_process.spawn` with `shell: true` invoking a `.exe`, it's the shelling-out pattern and is subject to the failure modes above.

- **PowerShell `where` is `Where-Object`, not `where.exe`.** When verifying installed binaries, use `where.exe <name>` or `Get-Command <name>`. Bare `where <name>` in PowerShell silently invokes the pipeline filter and returns nothing when there's no input, which looks identical to "binary not found." This cost about one round trip during the install verification phase of this arc and is worth keeping in muscle memory.

- **SSH-client config drift.** Termius maintains its own host database, separate from `~/.ssh/config`. When debugging an SSH issue, "Termius works" does not prove `~/.ssh/config` is correct — they're independent sources of truth. The canonical cross-check for `~/.ssh/config` correctness is PowerShell's native `ssh` (which DOES read `~/.ssh/config`). Use both Termius (proves server-side health) and PowerShell ssh (proves `~/.ssh/config` correctness) when triangulating.

## Related context

This entry, PEL-009, and PEL-011 together comprise a complete map of one SSH MCP install's lessons: install-time failure (PEL-009: SSL inspection), runtime failure (PEL-010: native-ssh wrapper fragility), and meta-pattern (PEL-011: Claude's own diagnostic blind spots that compound both). All three are atomic and self-contained, but reading them in order tells the story of how a single MCP setup required several distinct categories of debugging insight to actually ship.

The arc that produced this entry started with an SSP scoped for "wire SSH MCP, run two verification tests, draft PEL-009." It ended with the SSH MCP working via a different package than originally specified and three PEL entries rather than one — a useful prior for how to scope future MCP-install SSPs.
