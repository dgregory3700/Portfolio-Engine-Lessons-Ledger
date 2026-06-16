# OE-003 — EC2 Persistent Worker Online

## Date

2026-06-16

## Entry type

Opportunity Engine / Runtime Milestone

## Status

Completed.

## Summary

The isolated EC2 worker for the DuelTech Opportunity Engine is now running under PM2 and is configured to restart after EC2 reboot.

## Runtime

```text
Instance name: dueltech-opportunity-engine-worker
Public IPv4: 3.129.14.7
OS: Ubuntu 26.04 LTS
SSH user: ubuntu
Process manager: PM2
Service: pm2-ubuntu
```

## Verified

```text
PM2 worker status: online
PM2 process list: saved
systemd service: enabled
systemd service: active
```

## Safety boundary

The worker remains in connected-dry-run mode.

No real source discovery, outreach, paid model/search calls, production writes, HarmonyDesk infrastructure, or Entropy Cleaner infrastructure were touched.

## Current state

The Opportunity Engine now has a live isolated runtime foundation:

```text
GitHub control plane
+ isolated Supabase persistence
+ isolated EC2 worker
+ PM2 persistence/reboot survival
```

## Next action

Add health/status logging so future agents can verify worker state without manual terminal inspection.
