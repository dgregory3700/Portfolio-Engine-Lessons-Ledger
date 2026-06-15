# OE-000 — Opportunity Engine Control Plane Initialized

## Date

2026-06-14

## Entry type

Opportunity Engine / Control Plane

## Status

Active planning and build preparation.

## Summary

DuelTech is creating the Opportunity Engine: a persistent, stateful machine for finding and validating software opportunities without manual AI-to-AI copy/paste relay.

The engine's purpose is to improve DuelTech function #1:

```text
Find a good reason to build software.
```

This is separate from function #2, building software, which is already more agent-friendly when AI agents have access to GitHub, Vercel, Supabase, EC2, and related infrastructure.

## Core doctrine

The Opportunity Engine should not behave like a chat assistant that stops after each phase and asks whether to continue.

Default behavior:

```text
Continue until blocked, rejected, parked, advanced, or authorization is required.
```

The engine must perform both:

```text
pre-candidate discovery
→ candidate admission
→ deep candidate validation
```

The important correction is that extensive investigation must happen before something becomes a formal candidate.

## Canonical implementation repo

```text
dgregory3700/dueltech-opportunity-engine
```

That repo contains the architecture, protocols, prompts, schemas, SSP, handoff, local project ledger, and future implementation code.

## PEL folder boundary

Opportunity Engine entries belong in this folder:

```text
opportunity-engine/
```

They should not be mixed into the main `ledger/` folder unless specifically needed.

## Next operational step

Build Opportunity Engine v0:

- worker skeleton,
- Supabase data model,
- pre-candidate discovery flow,
- evidence normalization,
- first non-mutating test run,
- dashboard later.
