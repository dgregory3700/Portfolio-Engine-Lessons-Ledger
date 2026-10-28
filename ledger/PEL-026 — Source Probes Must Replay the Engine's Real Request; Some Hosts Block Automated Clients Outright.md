# PEL-026 — Source Probes Must Replay the Engine's Real Request; Some Hosts Block Automated Clients Outright

**Date:** 2026-10-28 · **Project:** Calla / DuelTech Opportunity Engine (goose01) · **Consolidates:** SSP-026, -048, -049, -051, -054, -055, -056, -057 candidates
**Tags:** _probing · user-agent · request headers · regulations.gov · eCFR · govinfo · reginfo_

## Quick Recognition Test
A candidate URL passes a curl or browser probe, then 403s, returns a stub, or never arrives inside the engine run. Or the reverse: curl 403s a host the engine reads fine.

## Root Cause
Hosts fingerprint the whole request, not only the URL. A default curl UA, a browser UA and the engine's UA plus full header set get different answers. epa.gov 403s curl's default UA but serves the engine UA. NMPF served curl carrying the engine UA string yet refused the engine's full header set. downloads.regulations.gov is a CloudFront block on automated clients: engine UA, a regs.gov API key (header or param) and even robots.txt all 403. eCFR blocks the engine UA outright.

## What NOT to Waste Time On
- Probing with curl defaults or a browser UA and treating 200 as a pass.
- Retrying downloads.regulations.gov with the API key; the key does not unlock it.
- UA spoofing to get through a block; the engine must read sources as itself (D-090).

## Canonical Fix Pattern
- Probe with the engine's own request: replay `buildSourceRequestHeaders` (UA `DuelTechOpportunityEngine/0.1 live-research-preview` plus the full header set), then the engine extractor on the body (PEL-025).
- Known admissible substitutes when the primary host blocks:
  - Federal Register notice → federalregister.gov full text, or govinfo.gov HTML/PDF.
  - CFR text → govinfo CFR XML (`CFR-YYYY-titleNN-volMM-secXXX-YY.xml`), which extracts through the HTML-strip path.
  - OMB Supporting Statement → reginfo `DownloadDocument` (objectID from the ICR page).
- Record every dropped host and why in the session record, so the next session does not re-probe it.

## When to Apply This Again
Before slating any new host, and any time a probe and a run disagree about the same URL.
