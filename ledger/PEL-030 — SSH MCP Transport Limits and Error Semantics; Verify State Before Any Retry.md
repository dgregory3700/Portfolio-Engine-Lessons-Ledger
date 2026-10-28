# PEL-030 — SSH MCP Transport Limits and Error Semantics; Verify State Before Any Retry

**Date:** 2026-10-28 · **Project:** Calla box over ssh-mcp (tufantunc/ssh-mcp, patched fork dgregory3700/ssh-mcp) · **Consolidates:** SSP-029, -031, -037, -046, -050, -053, -056 candidates
**Tags:** _SSH MCP · timeouts · exit codes · detached jobs · bridge drops_

## Quick Recognition Test
An SSH tool call errors, hangs or returns "No response from server", but the command may have done its work anyway.

## Root Cause
The transport's error signal is not the command's outcome:
- The default `--maxChars` capped commands at 1000 chars (fixed: `--maxChars=none`).
- Upstream rejected on stderr presence. The fork rejects on exit code and appends stderr under `[stderr]`.
- Exit-code rejection means a healthy `grep -c` with zero matches (exit 1) surfaces as a tool error with its output landed.
- Calls hard-timeout at 60 s.
- `nohup ... &` can hold the channel to that timeout even though the job started; `setsid nohup ... > log 2>&1 < /dev/null &` returns cleanly.
- `pkill -f` hangs the channel to the 60 s SIGTERM.
- Large heredocs (~3 KB+) returned "No response from server" and landed nothing.
- When the desktop bridge drops, detached box jobs keep running.

## What NOT to Waste Time On
- Retrying an errored call blindly; a chained git or gate step may already have happened.
- Building workarounds into doctrine before reading the tool's source; the cap and the stderr rejection were both config/code facts.

## Canonical Fix Pattern
- After any error or no-response, verify actual state first (`git log --oneline -1`, md5 of the gate file, `pgrep`, file sizes), then decide.
- Run anything near 60 s detached with `setsid nohup ... > log 2>&1 < /dev/null &`, then poll the log.
- Use `kill <pid>`, never `pkill -f`.
- End chains with a command that exits 0, or read zero-match grep errors as "no matches".
- Cross-check the box clock against GitHub's `Date` header at open; it drifted ~32 h once.

## When to Apply This Again
Every session on the Calla box, and any other ssh-mcp target.
