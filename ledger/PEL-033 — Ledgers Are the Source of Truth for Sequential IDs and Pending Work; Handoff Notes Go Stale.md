# PEL-033 — Ledgers Are the Source of Truth for Sequential IDs and Pending Work; Handoff Notes Go Stale

**Date:** 2026-10-28 · **Project:** Calla (goose01) and Portfolio-Engine-Lessons-Ledger · **Consolidates:** SSP-043, -054, -055 candidates
**Tags:** _D-numbers · PEL numbers · OE build-status numbers · handoff staleness_

## Quick Recognition Test
You are about to cite "the next D-number", OE number or PEL number from a handoff or memory, or scope work from a handoff's "open items" list.

## Root Cause
Handoffs and memory are snapshots, while ledgers and repos keep moving:
- A proposed D-062 collided with an existing D-062 (landed as D-063, correction disclosed).
- OE build-status reports skipped from OE-023 to OE-024 because two closes did not push.
- A handoff listed scout v2.3 "open ideas" that had already shipped in v2.2.

## What NOT to Waste Time On
Reconciling numbering after the fact.

## Canonical Fix Pattern
- Read the ledger tail (`grep "^## D-" ledger | tail -2`), the OE directory listing, or the PEL `ledger/` listing immediately before numbering anything.
- Read the target repo's log or header before scoping work a handoff describes as pending.

## When to Apply This Again
Every new D-entry, OE report, PEL entry, and any session that opens from a handoff.
