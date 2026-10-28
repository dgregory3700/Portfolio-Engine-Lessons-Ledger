# PEL-032 — pm2 stop Without pm2 save Is Not a Durable Stop

**Date:** 2026-10-28 · **Project:** Calla box (PM2 worker under systemd resurrect) · **Source:** SSP-031/032 candidate
**Tags:** _PM2 · systemd · resurrect · dump file_

## Quick Recognition Test
A worker you stopped is running again after a reboot or PM2 daemon restart.

## Root Cause
`pm2-<user>` systemd startup runs `pm2 resurrect` from the dump file. `pm2 stop` changes live state only. If the dump still records the process as online, resurrect revives it.

## What NOT to Waste Time On
Hunting for a cron or deploy hook that "restarted" the worker.

## Canonical Fix Pattern
- `pm2 stop <name> && pm2 save`.
- Verify both: `pm2 jlist` shows status stopped, and the dump file records status stopped.
- Include the dump check in session-open and session-close checklists.

## When to Apply This Again
Any PM2 process meant to stay down across reboots. This sits next to PEL-006/015/016 in the PM2 family.
