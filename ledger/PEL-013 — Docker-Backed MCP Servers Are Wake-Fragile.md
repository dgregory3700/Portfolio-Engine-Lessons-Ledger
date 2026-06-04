# PEL-013 — Docker-Backed MCP Servers Are Wake-Fragile (Stdio-Backed Ones Aren't)

**Date logged:** June 3, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop, MCP infrastructure)
**Related entries:** PEL-009 (Corporate SSL Inspection Silently Breaks Node MCP Attach) and PEL-010 (Native ssh.exe MCP Wrappers Fail Silently on Windows) cover install-time and runtime fragility for the stdio Node-binary MCP transport. PEL-013 covers post-resume fragility for the Docker MCP transport. Together the three PELs map MCP failure surfaces by transport family.

---

## Symptom

Mid-Cat-3F session, in an arc where both MCP servers (GitHub via Docker, SSH via Node binary at `C:\Users\dgreg\AppData\Roaming\npm\ssh-mcp.cmd`) had been operational for hours, the user shut down their laptop for approximately 8 hours. On restart, Claude Desktop fired two orange popup banners simultaneously:

> MCP github: Server disconnected. For troubleshooting guidance, please visit our debugging documentation
> Could not attach to MCP server github

Both MCPs were configured side-by-side in `claude_desktop_config.json`. Both had attached cleanly during the prior session. After resume, only one came back.

Probes from inside Claude confirmed asymmetric state:

- `github:get_me` → "Tool not found."
- `ssh:exec` with simple `whoami` → returned `ubuntu` cleanly within seconds.

Notably, Claude Desktop's developer-settings panel showed BOTH MCPs as "running" (blue badge) even while GitHub tool calls were failing — that asymmetry is documented separately in PEL-014.

## Root cause

MCP servers in Claude Desktop fall into transport families with materially different post-resume recovery characteristics.

**Stdio Node binary (SSH MCP via ssh-mcp.cmd).** Claude Desktop spawns the binary as a child process. On laptop wake, Claude Desktop's MCP attach process re-spawns the binary directly. The binary has no external service dependencies; it starts, connects to the configured SSH endpoint, and reports ready. The whole cycle completes within Claude Desktop's attach timeout because nothing has to coordinate with another desktop application.

**Docker-backed (GitHub MCP via `docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server`).** Claude Desktop's MCP attach process invokes `docker run` against the local Docker daemon. On laptop wake, two desktop applications are starting in parallel — Claude Desktop and Docker Desktop. Docker Desktop typically takes longer to fully initialize (it has to start its VM, restore the engine, accept commands). If Claude Desktop attempts to attach before Docker is ready to accept `docker run` invocations, the attach fails. Claude Desktop does not retry; the MCP is marked disconnected and stays disconnected until a manual restart of Claude Desktop after Docker is up.

The mechanism is a startup-ordering race between two independent desktop applications. The race is hidden in steady-state operation (everything eventually starts up and the system reaches its working configuration) but is exposed by every wake/resume cycle, where Claude Desktop frequently wins the race against Docker.

## Diagnostic technique

When MCP probes fail after a laptop wake, shutdown/restart, or hibernation cycle:

1. **Check Docker Desktop's system tray icon.** Is it fully running (engine icon stable), still spinning up (engine icon animated), or not started at all? Docker Desktop must be in "Engine running" state before Claude Desktop can attach Docker-backed MCPs.
2. **Open Claude Desktop's developer settings panel.** Note which MCP servers are listed and which transport family each uses (Command = `docker` indicates Docker-backed; Command = a `.cmd` or `.exe` indicates stdio). For Docker-backed MCPs, expect "disconnected" or attach-failed states after a sleep cycle even if the badge reads "running" (see PEL-014).
3. **Probe each MCP via its simplest tool call.** Badge and popup are inputs; probe is the truth.
4. **Resolution sequence**: fully quit Claude Desktop. Confirm Docker Desktop is in "Engine running" state. Restart Claude Desktop. Re-probe each MCP. Attach succeeds on the second startup because Docker is now ready.

## Canonical fix

There is no configuration-level fix — the race is structural between Claude Desktop's attach timing and Docker Desktop's startup timing. The remediation is procedural and lives at the user level:

**After any laptop sleep, shutdown, or hibernation:** treat Docker-backed MCPs as needing manual reattach. The procedure is:

1. Power on / wake laptop.
2. Confirm Docker Desktop fully initialized (tray icon: Engine running).
3. THEN start Claude Desktop, or if Claude Desktop auto-launched on boot, quit it and restart.
4. Probe each MCP via a simple tool call before relying on it.

**Within a single multi-arc work session:** if the user reports having stepped away ("just got back," "stepped away for X hours"), include an MCP probe check before assuming the prior session's tool state holds. Stdio MCPs typically survive resume cycles; Docker MCPs typically don't.

## Operational consequence

For the Cat 3F arc: ~3 turns of conversation were spent diagnosing the asymmetric MCP state, walking the user through Docker Desktop UI checks, then waiting for a Claude Desktop restart cycle. Net cost: ~10-15 minutes of arc time. Production was untouched throughout — the work was simply paused.

The broader operational consequence is that Docker-backed MCPs introduce a hidden recovery cost on laptop-based development workflows where sleep/shutdown is normal. Users who shut down their laptop at end-of-day will see this every morning when starting Claude Desktop. The friction is small per occurrence but constant. Stdio-backed MCPs do not introduce this friction.

## Recurrence prevention

- **Standing user-side procedure for laptop-based MCP workflows**: after wake or restart, start Claude Desktop AFTER confirming Docker Desktop's tray icon shows Engine running. If Claude Desktop was launched first (auto-start), quit it and restart once Docker is up.
- **Standing session-open gate**: probe each MCP via a minimal tool call before relying on it. Status badges and prior-session bridge claims are inputs, not conclusions. PEL-012 (bridge-doc claims) and PEL-014 (UI badge reliability) cover related signal-hygiene patterns.
- **Forward-looking install decision**: when there's a choice between stdio-transport and Docker-transport packaging for an MCP server, prefer stdio for wake-fragility reasons alone. The trade-offs vary by MCP (some only ship Docker images; some Docker installs are materially simpler than stdio for cross-platform reasons), but where there's a choice, stdio wins on resume reliability.

## Related context

This PEL completes the SSH MCP setup arc's lesson set. PEL-009 documented the install-time failure mode (corporate SSL inspection breaks Node MCP attach). PEL-010 documented the runtime failure mode (native ssh.exe wrapper fails silently on Windows). PEL-013 documents the post-resume failure mode (Docker-backed MCPs lose their attach on wake). The three together map three distinct failure surfaces — install, runtime, resume — and demonstrate that each MCP install's lessons span multiple phases of the tool's lifecycle.

The transport asymmetry observed in PEL-013 (Docker fragile, stdio resilient on wake) inverts the asymmetry observed in PEL-009 (Docker not affected by corporate SSL inspection because the image was pulled in a non-corporate-network context; Node MCP install affected). This is worth noting: the "right" transport choice depends on which failure mode you're optimizing against, and laptop-based development arcs tend to expose the wake-fragility one more often than the install-fragility one.
