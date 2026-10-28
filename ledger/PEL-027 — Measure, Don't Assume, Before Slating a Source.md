# PEL-027 — Measure, Don't Assume, Before Slating a Source

**Date:** 2026-10-28 · **Project:** Calla (goose01) · **Consolidates:** SSP-026, -029, -045, -050, -051, -053, -055 candidates
**Tags:** _probing · fetch timeout · char windows · extractor offsets · drift markers_

## Quick Recognition Test
A probe "passed", but the run times out on the source, a char window lands on the wrong text, a drift check calls a marker MISSING that is really there, or a vendor page extracts single-digit characters.

## Root Cause
Probe checks covered the wrong quantity: HTTP status instead of fetch time, recorded offsets instead of today's offsets, the expected shape of a string instead of its rendered form. Each assumption held on one day and failed on another.

## What NOT to Waste Time On
- Trusting a status-200 probe for a heavy PDF (a 2.34 MB ESA took ~18.9 s against the 10,000 ms `OE_FETCH_TIMEOUT_MS` default).
- Treating one fast fetch as retiring a measured slow fetch; keep the timeout the slow measurement justified.
- Reusing a char window from a previous session. Offsets drift between sessions (Illinois NetDMR 18.9K–20K → 18.1K–21.2K).
- Writing drift markers from memory ("$49.99" vs rendered "$ 49.99").

## Canonical Fix Pattern
- Time the fetch and compare it to the configured timeout; raise the timeout per heavy source.
- Re-run the engine extractor and re-measure offsets in the same session before any windowed slate.
- Dump the extracted text and copy markers from it verbatim.
- Check `totalExtractedChars` in diag. Single-digit counts on a 200 page with real HTML mean a region-picker defect (PEL-038).
- For a pain candidate, scan `OPERATOR_PAIN_ANCHORS` over the full extracted text first. If no anchor exists anywhere, no window can make it prove pain.
- Remember that a window can change categorization (RMA operator_pain → competitor_vendor once windowed onto pricing), which changes route eligibility. Diag the windowed config, not the unwindowed one.

## When to Apply This Again
Every slate, and every time a window, timeout or marker is reused from an earlier session.
