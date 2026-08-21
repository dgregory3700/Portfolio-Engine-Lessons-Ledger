# OE-017 — Build status at SSP-045 close (2026-08-21 UTC)

## Where Calla stands relative to "ready"
Engine is **built and battle-tested**; the constraint is **terrain, not capability**. Through PC-005 the engine has cleared V1-V6 mechanics, OCR, windowed extraction, per-URL char windows, price atomization, WR proximity, and admission scoring — suite 277, unchanged this session (no engine code touched). What Calla still lacks is a *shippable* opportunity: five candidate terrains have now been worked (PC-001..PC-005 closed/parked) and PC-006 is mid-hunt with three terrains set aside.

## This session (SSP-045) — stage-zero/stage-1 only
- Zero gated runs, gate CLOSED throughout, budget 0/4, no D-entries.
- Fresh scout v2.1 slice pulled (RUNDATE 21 AUG 2026, 805 survivors); committed to scout repo (db348e8).
- PC-006 candidates DESK-VERIFIED + set aside (all inadmissible):
  - Apparel labeling (FTC 3084-0101/0103) — Prong-1 fail (93-94% physical attach-label burden).
  - UST 2050-0068 — Prongs 1-2 strong, Prong-3 (interposition) fail: crowded mature commercial field.
  - Food labeling 0910-0381 — excluded on operator domain read.

## Sharpened doctrine (candidate for scout v2.2 + terrain-screen rule)
The binding constraint on a *passive-income* terrain is the **thin-incumbent wedge**, not burden mass. High burden + strong WTP predicts a crowded market (OSHA, FSMA, UST all died here). Lead RRP survived because its buyer was fragmented/low-salience and the field was thin. Future terrain screens must test Prong-3 (interposition) EARLY and hard — it is the cheapest kill and the usual one.

## What remains toward "ready"
1. A terrain that passes all three D-039 prongs INCLUDING a live thin-incumbent wedge (PC-006 hunt continues, thin-incumbent filter first).
2. Scout v2.2: doc-vs-physical signal (the pre-rank currently rewards the attach-label per-unit trap) + thin-incumbent proxy.
3. Standing engine backlog (non-blocking): selection-competition pattern, FR-PDF extractor defect, annex family-slot capture.

## End state
Gate CLOSED (32c1a811), PM2 stopped, budget 0/4 UTC 08-21, engine tree clean (HEAD = SSP-046 handoff commit 0ad5d14), scout clean (db348e8). Transport lessons carried in SSP-046 handoff (pkill-hangs-ssh-mcp; setsid orphans; box clock drift; .cjs for ad-hoc scout scripts; reginfo EPA IC-title garble).
