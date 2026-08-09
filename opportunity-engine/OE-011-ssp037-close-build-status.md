# OE-011 — Calla build status at SSP-037 close (2026-08-09)

## Where Calla stands relative to "ready"
Calla remains a working gated discovery-and-validation engine, not yet a product decision. PC-004 (EPCRA Tier II / TRI facility chemical reporting) is the live candidate: unconditionally ADMITTED (D-043/D-046) and now **V1 PASS** (D-052) — documentary burden quantification, the stage that parked PC-001 and PC-002, cleared on the already-admitted record (OMB 2070-0212, 22–36 hrs per response, TRI branch) at zero run cost. Validation ladder ratified: V2 data availability + work-removal next, then WTP. PC-003 stays PARKED, revival-eligible.

## Engine deltas this session
- D-051: annex seed-weighted ranking + per-family cap 2 — fixes the seventh selection-competition sighting (measured: a label-count tie resolved by earliest index alone). Suite 217/217, zero regressions.
- Two new filed items: OCR passage-segmentation defect (D-051 Ruling C); PRICE_LABEL_TERMS vocabulary gap = fourth vocabulary-trailing list family (amendment refused at n=1 under D-048 provenance).
- Hygiene: stale multi-session /tmp scratch found and cleared; close-checklist amended (full-listing sweep, prefix-at-creation discipline).

## What still needs doing (validation path)
1. PC-004 V2 data-availability arc — slate design around importable-input evidence (Tier2Submit/E-Plan import docs, NOAA data standard, state bulk-upload pages); the natural run to retire pending production confirmations (D-048, D-051, D-049 hit-bearing).
2. V3+ WTP after V2.
3. Plumbing that gates evidence classes: FR-PDF extractor defect; OCR segmentation defect; Tier II burden-figure fetch path (SS-A repair / reginfo post-OIRA).

Zero gated runs this session; budget 0/4 UTC 08-09 at close; gate CLOSED throughout; engine HEAD bb5d0bd.
