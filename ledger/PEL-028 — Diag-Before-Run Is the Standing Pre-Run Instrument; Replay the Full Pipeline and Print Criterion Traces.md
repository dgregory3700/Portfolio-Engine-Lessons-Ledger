# PEL-028 — Diag-Before-Run Is the Standing Pre-Run Instrument; Replay the Full Pipeline and Print Criterion Traces

**Date:** 2026-10-28 · **Project:** Calla (goose01) · **Consolidates:** SSP-032, -045, -049, -050, -051, -055, -058/059 candidates
**Tags:** _diag replay · admission gate · run budget · run isolation · OE_MAX_SOURCE_URLS_

## Quick Recognition Test
A budget-limited gated run returns a verdict nobody predicted, a slate source silently never appears in the report, or a correct verdict prediction hides a wrong reason.

## Root Cause
The engine's verdict comes from a chain (fetch → extract → categorize → summary → criterion predicates → gate), and several links have silent defaults:
- `OE_MAX_SOURCE_URLS` defaults to 3 and slices the slate tail pre-fetch with no log line.
- The gate scores each CLI invocation's dossier alone. Sources run in separate invocations never combine (run isolation, D-071).
- `runAdmissionGate` without `getSeedTerms(seed)` silently disables the D-073 WTP route.
- A run's invocation format is itself part of the configuration; a wrong format voided run 001045Z.

## What NOT to Waste Time On
- Predicting a verdict from the summary alone.
- Running single-URL probes and adding up their results as if they were one assessment.
- Trusting a matching verdict without its trace. In D-093 the diag predicted 3/5 exactly while both buyer legs were defective.

## Canonical Fix Pattern
- Replay the full chain through `adapter.discover()` (or the exported functions) over the exact slate, window and OCR config, then call `runAdmissionGate(..., getSeedTerms(seed))` exactly as `runEngine.ts` does.
- Print, per criterion, which evidence item carried it and by which route, not only the verdict.
- Set `OE_MAX_SOURCE_URLS` to the slate length and check the report's RS-LIVE count against the slate.
- For a candidate's true standing, run all its sources in one invocation.
- Diag the exact invocation string the run will use.

## When to Apply This Again
Before every gated run. The pattern predicted six consecutive production runs exactly.
