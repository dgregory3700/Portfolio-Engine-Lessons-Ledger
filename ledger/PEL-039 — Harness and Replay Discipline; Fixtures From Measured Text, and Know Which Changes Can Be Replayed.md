# PEL-039 — Harness and Replay Discipline; Fixtures From Measured Text, and Know Which Changes Can Be Replayed

**Date:** 2026-10-28 · **Project:** Calla engine changes (D-series) · **Consolidates:** SSP-041, -042, -043, -049, -050, -054, -057, -059 candidates
**Tags:** _test harness · fixtures · consequence replay · rank sentinels_

## Quick Recognition Test
A new harness case fails, or a change is about to land with a "consequence replay" claim.

## Root Cause
Several patterns recurred:
- Self-authored expectations are the usual failure. In D-061 two test cases were wrong, not the design. In D-097 a fixture asserted the ACO kit string was a price artifact; pre-change code showed it never was.
- Harnesses surface second-order defects (post-decode whitespace, abbreviation splitting).
- Exported reports keep summaries but not delivered text or which route fired. Gate-level changes can be replayed over the corpus; extractor and categorizer changes cannot.
- An exported-function replay (`inferSourceCategory`, `createSummary`, `applySourceWindow`, criterion predicates) reproduces the engine profile offline with zero network.

## What NOT to Waste Time On
- Weakening a failing assertion before checking whether the fixture or the code is wrong.
- Claiming a corpus replay for a categorizer change.

## Canonical Fix Pattern
- Build fixtures from verbatim measured text.
- When a case fails, run the same input through the pre-change code. A false-before-and-after result means the fixture is wrong; re-pin it as unchanged and disclose.
- Keep rank sentinels (known-good shapes logged every run) for any heuristic change.
- State the replay limit in the D-entry when a change cannot be replayed.
- Measure categorizer and summary questions with offline exported-function replays, or cache-only replays of a prior scout run.

## When to Apply This Again
Every engine change.
