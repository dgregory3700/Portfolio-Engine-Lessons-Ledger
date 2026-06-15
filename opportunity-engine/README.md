# Opportunity Engine Ledger Area

This folder is reserved for DuelTech Opportunity Engine material.

It is intentionally separate from the canonical `ledger/` folder.

## Purpose

This area gives AI agents a clean, unambiguous place to read and write Opportunity Engine status, doctrine, milestones, and handoff references without confusing those entries with Claude's existing software-build ledger entries.

## Boundary

Use this folder for:

- Opportunity Engine milestones,
- Opportunity Engine doctrine updates,
- machine architecture decisions,
- cross-agent handoff references,
- links to the `dueltech-opportunity-engine` repo,
- high-level status summaries.

Do not use this folder for:

- raw evidence dumps,
- every rejected raw signal,
- every validation check,
- long research reports,
- unrelated software-build troubleshooting entries.

Detailed Opportunity Engine state belongs in the `dueltech-opportunity-engine` repo and its future Supabase database.

## Canonical project repo

The implementation/control-plane repo is:

```text
dgregory3700/dueltech-opportunity-engine
```

Agents working on the Opportunity Engine should read that repo first.

## Agent instruction

When an agent is working on the Opportunity Engine:

1. Read this folder for cross-agent status.
2. Read the `dueltech-opportunity-engine` repo for protocols, code, schemas, prompts, SSPs, and handoffs.
3. Do not modify the main `ledger/` folder for Opportunity Engine work unless explicitly instructed.
4. Keep entries here concise, operational, and clearly labeled as Opportunity Engine material.
