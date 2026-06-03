# PEL-009 — Corporate SSL Inspection Silently Breaks Node MCP Attach

**Date logged:** June 2, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop + Node-based MCP servers)
**Related entries:** PEL-008 (Silent mcpServers Block Deletion in Claude Desktop Config). Adjacent infrastructure layer — same `claude_desktop_config.json`, same Claude Desktop attach cycle, same class of "silent failure during MCP attach." PEL-008 was about a missing block; PEL-009 is about a present block whose runtime fetch fails invisibly. PEL-010 and PEL-011 are queued from the same arc and will cover the native-ssh-spawn failure mode and Claude's own diagnostic blind spots respectively; this entry is self-contained without them.

---

## Symptom

A new MCP server entry is added to `%APPDATA%\Claude\claude_desktop_config.json` using the standard `npx`-at-runtime pattern published in the package author's install docs:

```json
"ssh": {
  "command": "cmd",
  "args": ["/c", "npx", "-y", "@aiondadotcom/mcp-ssh"]
}
```

After Claude Desktop is fully quit from the system tray and reopened, a toast popup fires in the top-right corner:

> ⚠ Could not attach to MCP server ssh

Claude Desktop continues running normally. Other MCP servers (e.g., GitHub via Docker) attach without issue. The new server's tools never appear in any session's tool catalog. There is no error in the user-visible UI beyond the single toast, which auto-dismisses in seconds.

Inside `%APPDATA%\Claude\logs\mcp-server-ssh.log`, the spawned child writes:

```
npm error code UNABLE_TO_VERIFY_LEAF_SIGNATURE
npm error errno UNABLE_TO_VERIFY_LEAF_SIGNATURE
npm error request to https://registry.npmjs.org/<pkg> failed, reason: unable to verify the first certificate; if the root CA is installed locally, try running Node.js with --use-system-ca
```

Followed ~60 seconds later by `McpError: MCP error -32001: Request timed out` from the client side and `Server transport closed`.

On the same laptop at the same moment, `ssh`, `git clone`, every browser, and Docker pulls work normally against the same network.

## Root cause

The laptop is on a network that performs SSL inspection — a corporate or enterprise security proxy that terminates outbound TLS, inspects payload, and re-encrypts using a corporate-issued root CA installed in the Windows certificate store. On managed Windows laptops this is common; the giveaway in PATH is something like `C:\Program Files\HP\HP One Agent` or analogous enterprise management tooling.

Most tools on Windows trust the corporate root CA automatically because they read the Windows certificate store: ssh.exe, git, every browser, Docker Desktop's TLS stack. They see the corporate-signed leaf cert as valid because the corporate root is trusted at the OS level.

**Node.js does not.** Node ships its own CA bundle compiled into the binary and by default does not read the Windows certificate store. When `npm` (which runs on Node) talks to `registry.npmjs.org`, it sees a cert chain signed by a root it doesn't recognize, fails verification, and exits with `UNABLE_TO_VERIFY_LEAF_SIGNATURE`. The error message itself hints at the fix ("try running Node.js with `--use-system-ca`") but this requires Node 22+ and intercepting how npm invokes Node, which is not viable from inside a static MCP config block.

The MCP-specific consequence: every Node-based MCP server published using the `npx -y <package>` pattern in its install docs assumes `npm` can reach the registry at MCP attach time to resolve, download, and cache the package on first run. On a corporate-SSL-inspection laptop, the registry fetch fails inside the Claude-Desktop-spawned child process, the MCP server process never completes its handshake, and Claude Desktop's 60-second attach timeout fires. The popup is the only user-visible signal. The actual `UNABLE_TO_VERIFY_LEAF_SIGNATURE` line is buried in the per-server log file most users never open.

## Diagnostic technique

Recognition order, fastest first:

1. **PATH inspection.** If `C:\Program Files\HP\HP One Agent` (or any other enterprise management/agent path) appears in `%PATH%`, assume SSL inspection is in play. Confirm by running `npm ping` from PowerShell — if it returns `UNABLE_TO_VERIFY_LEAF_SIGNATURE`, the diagnosis is locked. If `npm ping` succeeds, the issue is elsewhere.
2. **Per-server MCP log file**: `%APPDATA%\Claude\logs\mcp-server-<name>.log`. The npm cert error lines appear verbatim. Search for `UNABLE_TO_VERIFY_LEAF_SIGNATURE` or `unable to verify the first certificate`.
3. **Cross-tool sanity check.** Confirm `ssh`, `git fetch`, or a curl from PowerShell to the same registry URL works. If they do but Node tooling doesn't, the diagnosis is locked. If they all fail, the network itself is blocking and the problem is upstream of cert handling.

What is *not* diagnostic, despite intuition suggesting otherwise:

- The Claude Desktop popup itself, beyond confirming attach failed. The popup is identical regardless of which kind of failure occurred (cert error, JSON syntax error, missing binary, timeout, crash).
- Docker Desktop's state. Containers and images are unrelated to a Node-based MCP server's npm registry fetch.
- The npm cache state at `%LOCALAPPDATA%\npm-cache`. Even with a stale or empty cache, the bug is cert verification, not cache freshness.
- Restarting Claude Desktop or Docker. Neither restart changes Node's CA bundle.

## Canonical fix

Take `npx` out of the MCP attach runtime path entirely by pre-installing the package globally **once**, from an interactive PowerShell:

```
npm install -g <package>
```

Empirically on this laptop, `npm install -g` from interactive PowerShell succeeds despite the SSL inspection environment, while `npx` from a Claude-Desktop-spawned child process fails with the cert error. The exact mechanism for this difference is not fully traced — candidates include npm cache state, npm config inheritance, env var propagation (`NODE_EXTRA_CA_CERTS`, `HTTPS_PROXY`), or differing TLS code paths between `npm install` and `npx` — but the observable behavior is reliable across multiple test cycles. If a future `npm install -g` also returns the cert error, the immediate workaround is `npm config set strict-ssl false`, run the install, then `npm config set strict-ssl true` again. This is lower-security posture and should never be left enabled.

After the global install, change the MCP config block from the `npx`-at-runtime pattern to an explicit absolute path to the installed binary:

```json
"ssh": {
  "command": "C:\\Users\\<user>\\AppData\\Roaming\\npm\\<binary>.cmd",
  "args": [...]
}
```

Confirm the binary path after install with `where.exe <binary>` (note: `where.exe`, not `where` — PowerShell aliases `where` to `Where-Object`, which silently returns nothing instead of searching for the binary). Paste the returned path into the config.

Three reasons absolute paths are preferable to bare binary names even after a successful global install:

1. The npm global directory may or may not be on the PATH that Claude Desktop's child-process spawn inherits. Interactive PowerShell's PATH and the spawned-child PATH are not guaranteed equivalent.
2. Absolute paths eliminate ambiguity about *which* `<binary>.cmd` is being resolved if multiple versions exist on disk.
3. Future debugging is faster because the exact command being spawned is visible in `mcp-server-*.log` and can be replicated manually.

## Operational consequence

The failure is silent in three layers: Claude Desktop (one toast that disappears in seconds), the MCP server (its log file exists but is rarely opened), and Node itself (the cert error message is technically correct but located several levels deep in a file most users never visit). A user without familiarity with corporate-SSL-inspection environments will spend hours chasing the wrong layer — verifying their config syntax, reinstalling Node, reinstalling Claude Desktop, exploring Docker Desktop, regenerating the MCP package from GitHub, swapping MCP packages, even (as happened in the arc that produced this entry) believing the problem lies downstream of MCP attach when the MCP never attached in the first place.

A secondary cost: every Node-based MCP package's published install docs assume `npx`-at-runtime is the canonical pattern. The corporate-laptop user reading those docs has no upstream signal that they need to deviate — the docs work fine on the package author's laptop, on the package author's CI, and on most consumer-grade users' machines. The pattern fails specifically and silently on managed/enterprise environments, which are a high-value user segment because they're often the ones with the highest-value internal infrastructure to integrate.

## Recurrence prevention

- **Standing rule for this laptop: always `npm install -g <pkg>` from interactive PowerShell before adding a Node-based MCP server to `claude_desktop_config.json`.** Never paste a `"command": "npx"` block into the config and trust that npx will resolve at runtime. This applies to the future Filesystem MCP install, any future SSH MCP swap, and every other Node-based MCP. Docker-based MCPs (e.g., GitHub MCP via `ghcr.io/github/github-mcp-server`) are immune because Docker Desktop's image fetch uses its own TLS stack which trusts the Windows certificate store correctly.
- **Add `npm ping` to the install playbook.** Before any `npm install -g`, run `npm ping` to confirm the registry is reachable from the current shell. If it returns the cert error, the user knows immediately they're hitting this PEL and can take corrective action upfront rather than discovering it via a silent MCP attach failure ten minutes later.
- **Treat MCP install-time secrets as already burned the moment they appear in a chat surface.** Adjacent lesson from the same arc: a `GITHUB_PERSONAL_ACCESS_TOKEN` value was pasted into a Claude chat for "show me the current config file" purposes. Plaintext secrets in `claude_desktop_config.json` are one risk; the same secrets pasted into any AI chat surface for clarity purposes are a different risk with the same blast radius. The takeaway: rotate immediately, do not paste the rotated value back into chat, and add the MCP config file to the same rotation cadence as `STRIPE_WEBHOOK_SECRET`, `APP_TOKEN_SECRET`, `RESEND_API_KEY`, `SUPABASE_SERVICE_ROLE_KEY` (the post-Cat-2H security hygiene queue).

## Related context

This entry was logged after a multi-hour SSH MCP setup arc in which `UNABLE_TO_VERIFY_LEAF_SIGNATURE` was the first of three distinct failure modes the SSH MCP layer surfaced before reaching a verified state. The other two — native-`ssh.exe`-spawn issues on Windows from inside a Claude Desktop child process, and Claude's own introspection blind spots when MCP attachment fails — are captured separately in PEL-010 and PEL-011 because each meets the atomic-and-self-contained criterion on its own. If you are reading PEL-009 because you've just hit `UNABLE_TO_VERIFY_LEAF_SIGNATURE` adding a new Node-based MCP and you're on a corporate-managed laptop, the fix above is sufficient. You do not need to read PEL-010 or PEL-011 unless your subsequent attach also fails.
