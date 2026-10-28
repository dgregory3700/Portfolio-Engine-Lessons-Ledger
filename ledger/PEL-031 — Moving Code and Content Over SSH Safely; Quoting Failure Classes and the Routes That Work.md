# PEL-031 — Moving Code and Content Over SSH Safely; Quoting Failure Classes and the Routes That Work

**Date:** 2026-10-28 · **Project:** Calla box over ssh-mcp · **Consolidates:** SSP-030, -036, -037, -038, -041, -042, -043, -050, -051, -052, -053, -055, -057, -058 candidates
**Tags:** _SSH MCP · bash -lc quoting · heredoc · base64 · patchers · scratch hygiene_

## Quick Recognition Test
A file written over SSH lands wrong with no error, a command breaks on its own content, or a scratch script fails to import engine modules.

## Root Cause
Content passes through two shells (`bash -lc '...'` inside the tool's command string). Measured failure classes:
- An apostrophe inside a single-quoted `bash -lc` ends the quote (observed 5+ times).
- A heredoc inside a double-quoted `bash -lc` silently strips inner double quotes. It landed wrong content with no error; tsc caught it.
- A heredoc whose terminator is the last line of the command fails; follow it with another command.
- JSON in a single-quoted command gets brace-expanded.
- Heredoc Python inside single quotes breaks on any single quote; f-strings with subscripts are the usual trigger.
- Hand-copied base64 chunks can be pasted from the wrong buffer.
- Scratch scripts in /tmp fail module-relative imports.

## What NOT to Waste Time On
- Escaping your way through nested quotes.
- Trusting a transfer without a checksum.

## Canonical Fix Pattern
Ranked routes:
1. Author files locally or through the GitHub API, push, then `git pull` on the box. Zero quoting exposure.
2. gzip → base64 → decode on box, with md5 of the decoded file checked against the source. Split into labelled chunks only when needed, md5 each chunk, and retransfer on mismatch.
3. A small, apostrophe-free Python exact-anchor patcher on the box that asserts each anchor matches exactly once.

Hygiene:
- Put scratch scripts at the repo root (or beside the module they import).
- Give every scratch file the `sspNNN_` / `tmp_sspNNN_` prefix at creation, including one-off curl outputs.
- At close, sweep with a full `/tmp` listing, not a prefix grep.

## When to Apply This Again
Any code or content change on a remote box through a shell-wrapping tool.
