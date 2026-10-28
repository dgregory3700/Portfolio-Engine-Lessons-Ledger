# PEL-040 — Gated-Run Runner Safety; Restore the Gate on Any Exit and Verify From the Log

**Date:** 2026-10-28 · **Project:** Calla gated live-research runs · **Consolidates:** SSP-037, -039, -040, -053 candidates
**Tags:** _gate · atomic open/run/close · md5 · run budget · run reports_

## Quick Recognition Test
You are about to open the gate for a budgeted run, or reading results from a multi-source run report.

## Root Cause
Each of these traps was hit:
- A run left the gate open on an unexpected exit.
- A fast run restored the gate before a same-call verification grep, so the grep "failed" on a healthy run.
- A session straddled UTC midnight and counted the wrong budget day.
- A first-match grep misattributed Extraction lines, because a URL appears as a citation inside another source block before its own.
- `reports/` and `decision-ledger/drafts/` are gitignored. Only `reviewed-runs/` exports and the ledger are committed, so a `git add` on ignored paths fails.

## What NOT to Waste Time On
Copying an open-gate file to the box, or treating a post-launch grep miss as a failed run.

## Canonical Fix Pattern
- Use a detached, trap-guarded runner that restores the closed gate on any exit.
- Build the open-gate file from the closed one byte-exact (flag true plus `live-research-preview` in allowedModes) and verify md5 before opening and after closing; abort on any mismatch.
- Check the budget day against UTC at run time.
- Read the run log for the outcome.
- Parse reports one source block at a time.
- Commit only `reviewed-runs/` exports and ledger entries.

## When to Apply This Again
Every gated run.
