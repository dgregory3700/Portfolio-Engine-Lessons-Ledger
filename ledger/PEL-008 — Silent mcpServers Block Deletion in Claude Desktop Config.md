# PEL-008 — Silent mcpServers Block Deletion in Claude Desktop Config

**Date logged:** June 2, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop + Docker MCP)
**Related entries:** None directly — first ledger entry on the AI tooling infrastructure layer.

---

## Symptom

After previously working sessions where Claude Desktop had access to the GitHub MCP server (read/write on DuelTech repos), the GitHub tools silently disappeared. The user-visible signal was that Claude in Claude Desktop responded as if it had no GitHub awareness — it could not fetch repo contents, list issues, or commit files. There was:

- No error popup in Claude Desktop
- No error indicator in Docker Desktop
- The Docker image `ghcr.io/github/github-mcp-server:latest` still present in the local Images list
- Zero containers visible in the Docker Desktop Containers tab

The failure mode was pure absence — not a thrown error, just missing capability. A separately-consulted AI assistant inside Docker Desktop misdiagnosed this as "Claude hasn't activated the container yet," which delayed discovery of the real cause.

## Root cause

During an editing session of `%APPDATA%\Claude\claude_desktop_config.json`, the entire top-level `mcpServers` key block was deleted. The post-edit file was syntactically valid JSON containing only:

```json
{
  "preferences": { ... },
  "coworkUserFilesPath": "C:\\Users\\dgreg\\Claude"
}
```

Claude Desktop launches normally against a config with no `mcpServers` key — it simply has no MCP-based tools to spawn. There is no schema validation that flags the missing tooling block, and no warning surfaced to the user. The Docker image lingering in the Images tab was a red herring: having the image present is unrelated to anything actually launching the container. Containers are spawned only when Claude Desktop reads an `mcpServers` entry that instructs it to run `docker run -i --rm ... ghcr.io/github/github-mcp-server` at startup.

## Diagnostic technique

The recognition pattern, in order of speed:

1. **Open the config file directly** at `%APPDATA%\Claude\claude_desktop_config.json` and grep (or visually scan) for the literal string `mcpServers`. If absent, the bug is found.
2. **Confirming cross-signal:** open Docker Desktop's Containers tab. If empty while Docker Desktop is running and Claude Desktop is open, no MCP servers are being spawned — consistent with the config block being absent.
3. **Tertiary signal:** Claude's tool catalog within a Claude Desktop session does not contain any `github:*` tools (or other MCP-provided tools). The catalog reflects what was loaded at session startup based on the config.

What is not diagnostic, despite intuition suggesting otherwise:

- The presence of the Docker image in the Images tab (image present ≠ container running).
- Docker Desktop's MCP Toolkit (beta) UI showing empty profiles. The MCP Toolkit is a separate code path from the direct `claude_desktop_config.json` flow. An empty Toolkit is not evidence that the direct-config path is broken.
- Restarting Claude Desktop or Docker Desktop. Neither restart fixes the absence of the config block; it only surfaces the same empty state faster.

## Canonical fix

Restore the `mcpServers` block via **full-file replacement** of `claude_desktop_config.json`, preserving every previously-present top-level key (`preferences`, `coworkUserFilesPath`, and any others — these are not optional to retain). The block for the GitHub MCP server, per GitHub's official installation guide:

```json
"mcpServers": {
  "github": {
    "command": "docker",
    "args": [
      "run",
      "-i",
      "--rm",
      "-e",
      "GITHUB_PERSONAL_ACCESS_TOKEN",
      "ghcr.io/github/github-mcp-server"
    ],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "<PAT>"
    }
  }
}
```

After saving, Claude Desktop must be **fully quit from the system tray** (right-click the tray icon → Quit). Closing the window does not reload the config — the app continues running in the tray and will retain the stale (empty) tool catalog until terminated. After quitting, confirm Docker Desktop is running, then reopen Claude Desktop. The GitHub MCP container should appear in Docker Desktop's Containers tab within seconds of Claude Desktop startup.

## Operational consequence

The failure is silent in three layers — Claude Desktop, Docker Desktop, and the config file itself. None of them tell the user anything is wrong. The only positive signal that something is broken is the absence of expected tools when the user tries to use them, which means the bug can persist undiscovered until the user happens to ask Claude to do something GitHub-related and gets a flat "I don't have those tools" response.

A secondary cost: a user without familiarity with how MCP servers spawn (image vs. container, config-driven vs. UI-driven, stdio transport semantics) is prone to chase the wrong fix — reinstalling Claude Desktop, re-pulling the Docker image, or exploring the MCP Toolkit beta UI. These all consume time and leave the underlying config gap untouched.

## Recurrence prevention

- **Backup before edit.** Before any change to `claude_desktop_config.json`, copy the current file to `claude_desktop_config.json.bak` in the same directory. Restoration becomes a one-command operation if any edit removes content unexpectedly.
- **Verify presence after save.** After saving any edit, immediately re-open the file and visually confirm the `mcpServers` key is present at the top level alongside `preferences`. This takes five seconds and catches the failure at the source.
- **Full-file replacement only.** Treat the config as governed infrastructure under the same rule as production code — full-file replacement, never partial in-place edits that risk dropping unrelated keys. When an AI agent produces a replacement, the produced file must contain every previously-present top-level key explicitly.
- **Rotate the PAT under the same cadence as other secrets.** The `GITHUB_PERSONAL_ACCESS_TOKEN` value lives in plaintext in this config file. It belongs in the same rotation schedule as `STRIPE_WEBHOOK_SECRET`, `APP_TOKEN_SECRET`, `RESEND_API_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` (the post-Cat-2H security hygiene queue). Any breach of the laptop's user profile would expose this token; treat it accordingly.
- **Trust signals over narration.** External AI assistants (e.g., the Docker Desktop in-app AI) may produce confident-sounding explanations for an empty state ("Claude hasn't activated it yet") that are wrong. The decisive signals are the config file contents and the Docker Containers tab — not what another AI says about them.

## Related context

This entry was logged as the write-verification half of restoring GitHub MCP from a `claude_desktop_config.json` that had been stripped of its `mcpServers` block. The same session was the first round-trip GitHub MCP write from Claude Desktop after restoration — i.e., this file is simultaneously the lesson captured and the proof that the canonical fix worked. If you are reading this file in the GitHub UI, the round-trip is confirmed end-to-end.
