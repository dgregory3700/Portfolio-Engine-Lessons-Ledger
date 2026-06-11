# PEL-021 — A Node MCP Package's `.cmd` Shim Can Be Named Differently From the Package; Resolve the Real Binary Before Trusting `where.exe`

**Date logged:** June 10, 2026
**Project:** DuelTech AI tooling layer — Filesystem MCP install (Cat 3F-1 deck-clear, second item)
**Related entries:** PEL-009 (the canonical fix this builds on — pre-install globally, point config at the absolute `.cmd` path, never npx-at-runtime on the SSL-inspected laptop). PEL-014 (badge is not authoritative; probe with a real tool call). This entry is the missing sub-step inside PEL-009's "confirm the binary path with `where.exe <binary>`" instruction: it covers what to do when `where.exe` returns nothing.

---

## Symptom / context

Installing `@modelcontextprotocol/server-filesystem` followed PEL-009 exactly: `npm install -g` succeeded from interactive PowerShell (`npm list -g` confirmed `@modelcontextprotocol/server-filesystem@2026.1.14` in `C:\Users\dgreg\AppData\Roaming\npm`). The first config attempt still used the npx pattern and produced the expected PEL-009 popup: "Could not attach to MCP server filesystem" (github + ssh attached fine — only filesystem failed). Moving to PEL-009's direct-binary fix, the natural next command — `where.exe server-filesystem` — returned `INFO: Could not find files for the given pattern(s)`. This is the trap: the package IS installed, but `where.exe` searching for the *package* name finds nothing, which falsely reads as "no binary exists / install is broken."

## Root cause / mechanism

The npm-generated command shim is named after the package's `bin` entry, **not** the package name. For `@modelcontextprotocol/server-filesystem`, the npm scope (`@modelcontextprotocol/`) is stripped and the `bin` key produced a shim named **`mcp-server-filesystem.cmd`** — not `server-filesystem.cmd`. So `where.exe server-filesystem` searches for a binary that, by that name, does not exist; the actual shim is one `mcp-` prefix away. The package-name-vs-binary-name divergence is the same class of gotcha as PEL-020 (env var lives on the surface that reads it, not the one it names) and PEL-009's own SSH case (the `@aiondadotcom/mcp-ssh` package produced an `ssh-mcp.cmd` shim — also reordered/renamed).

## Diagnostic technique / verification

When `where.exe <package-name>` returns nothing after a confirmed global install, **list the npm global shim directory directly** instead of guessing the name:

```
dir "C:\Users\dgreg\AppData\Roaming\npm\*.cmd"
```

This prints every `.cmd` shim npm has created. The filesystem shim was plainly visible as `mcp-server-filesystem.cmd` (timestamp matching the install). Cross-reference: the ssh shim in the same listing is `ssh-mcp.cmd`, confirming the rename pattern is npm-wide, not package-specific. Once the real shim name is known, PEL-009's absolute-path fix applies verbatim.

## Canonical fix / pattern

Config block that attached cleanly (no popup; probe-verified by writing `hello.txt`):

```json
"filesystem": {
  "command": "C:\\Users\\dgreg\\AppData\\Roaming\\npm\\mcp-server-filesystem.cmd",
  "args": ["C:\\Users\\dgreg\\DuelTech"]
}
```

Note vs. the npx pattern: `command` is the absolute `.cmd` path; the npx-only args (`-y` and the package name) are dropped — the binary takes only its own arguments (here, the single allowed-root path). Restart Claude Desktop fully from the tray, then verify with a real write probe (PEL-014), not the badge: "create hello.txt in the allowed folder saying working." File appeared at `C:\Users\dgreg\DuelTech\hello.txt`, 7 bytes — live and write-capable.

## What NOT to waste time on

- Concluding the install failed because `where.exe <package-name>` found nothing. The install is fine; the shim just isn't named after the package. `dir *.cmd` settles it in one command.
- Re-running `npm install -g`, clearing the npm cache, or reinstalling Node. None of these is the issue — the binary already exists under a different name.
- `where` instead of `where.exe` (PEL-009 already flags this — PowerShell aliases `where` to `Where-Object` and returns nothing silently). Even with `where.exe`, the package-name search fails here for the *separate* naming reason above.

## When to apply again

- Every future Node-based MCP install on this laptop where the direct-binary path is needed and `where.exe <package-name>` comes up empty. Standing rule: after global install, run `dir "%APPDATA%\npm\*.cmd"` to read the real shim name rather than assuming it matches the package.
- Filesystem MCP scope note (durable): the server is sandboxed to `C:\Users\dgreg\DuelTech` and its subdirectories only. Any file deliverable a Desktop session is meant to write must land inside that tree. The Filesystem MCP attaches to Claude Desktop only — not claude.ai web.
