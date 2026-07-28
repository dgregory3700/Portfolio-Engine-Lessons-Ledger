# OE-009 — SSP-030 close build status

**Date:** 2026-07-27 (UTC)
**Repo state:** dgregory3700/goose01 @ `530e7c2` (HEAD == origin/main)
**Session:** SSP-030 — OCR-gap arc, PC-003 build decision, PC-004 intake

## Where Calla stands relative to "ready"

Calla is a working, gated discovery engine with 40 ratified decisions, 151 passing tests, and three candidates taken to a verdict. It is **ready as an instrument** and **unproven as a business input** — it has produced zero candidates worth building on, which is a real result rather than a failure of the tool.

## What landed this session

- **D-038 — OCR tier 3** (commit `0cffbcd`). Opt-in per-URL OCR for pure-scan PDFs (`OE_SOURCE_OCR`, `<url>=all` or page range). Reached only when both text tiers fail. Ruling A2: OCR text is admissible **only** with an `ocr` provenance block plus a report disclosure line stating it is a machine transcription of page images. Async (`execFile`), page range is the sole pre-run control, graceful degrade when poppler/tesseract are absent. New harness `ocr_test_run16.ts` 36/36; **suite 151/151**, zero regressions.
  - Fidelity was validated against independent ground truth: the transcript reproduced the D-028 operator-verified Cartersville record exactly — purchase amount, annual support figure, budget account, both council names, unanimous vote — at 94.4% mean confidence. Fixture committed permanently.
  - Closes an evidence-class gap open since SSP-025: scanned public records extracted to zero characters and were structurally invisible to the engine.

- **D-039 — terrain pre-screen doctrine** (commit `e900105`). Three-prong survivability filter, now a precondition for terrain ratification. Prong 1: obligation must be governed by a federal information collection (OMB control number, published burden estimates) — derived from PC-001 and PC-002 both clearing the admission gate 5/5 and then dying at V1 on state/local obligations that generate no PRA filing. Prong 2: buyer class must be private with observable self-serve behaviour — derived from PC-003's V5 PARK on a municipal buyer. Prong 3: check whether a free or mandated government tool occupies the base case, and require the wedge hypothesis to be written down in advance.
  - Binding limitation recorded: pre-screen findings are **evidence-inadmissible**, confer no credit, and shortcut nothing.

- **D-040 — PC-004 terrain** (commit `e900105`). EPCRA Tier II (§311/312) + TRI (§313) facility chemical reporting, the first terrain selected under a pre-screen rather than by judgment alone. Wedge hypothesis on record in advance: the paying pain is **assembly, not submission**. Risk recorded at selection: that wedge concentrates in multi-facility filers where funded incumbents already sit.

## Operator decision on PC-003

The operator asked whether a product could be built on the parked PC-003 candidate, considered the assessment, and **declined**. PC-003 stays PARKED, revival-eligible, and unbuilt. The reasoning is worth preserving: V1–V4 and V6 were well-evidenced, but V5 — the acquisition path — is the revenue mechanism, and municipal compliance tooling is not portfolio-shaped work for a solo founder.

## What still needs doing

1. **PC-004 five-criteria admission gate assessment** — next work item.
2. **Selection-competition pattern** — fourth observed instance, still unratified (D-030). Evidence has now twice reached extracted text and lost the three-slot summary competition.
3. **Cartersville OCR slate** — D-038 built the capability; the Item 15 vote is still **not** admitted evidence and needs its own authorized slate and gated run.
4. **PDF table-structure fidelity** — defect #4/#5 remnant, still open.

## Honest read

Two of the three decisions this session were doctrine, not code — the engine is increasingly limited by judgment about where to hunt rather than by extraction capability. D-039 is the first attempt to make terrain selection systematic instead of intuitive, and it immediately caught a risk (free federal tooling) that would otherwise have surfaced at V6 after an entire arc had been spent.

## Safety properties held

Gate CLOSED throughout and verified at close; PM2 stopped; **zero gated runs**; run budget 0/4 for UTC 2026-07-27; one operator-authorized probe. Scratch fully cleared. Tree clean except the standing untracked `package-lock.json`.
