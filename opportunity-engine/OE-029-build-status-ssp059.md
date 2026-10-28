# OE-029 -- build status at SSP-059 close (2026-10-28 UTC)

**Where Calla stands:** no candidate is active. This session was an engine-quality and housekeeping session with zero gated runs. Both remaining defects from the PC-008 admission record are now fixed and tested, so the buyer and price legs the next terrain relies on are cleaner. The lessons backlog that had accumulated since July is written.

## Shipped this session
- **D-096 (engine code):** the last-resort buyer fallback no longer fires on site-chrome words. Measured cause: the USAC de minimis page reached buyer_evidence through one word, "Procurement", in its footer nav. Fallback terms and the procurement role-guess branch now count only inside sentence-length passages. Structural, no vocabulary change.
- **D-097 (engine code):** penalty passages are no longer price artifacts. The per-unit price pattern read "$100,000 per violation" as a price, which put an FCC forfeiture line on the PC-008 citable summary and in the annex. The D-095 penalty lexicon now applies at the shared predicate, covering the summary bonus, the price-slot reservation and the annex. On replay, Performance West's real $1,499/yr tier takes the summary price slot.
- **Verification:** new harness run34 20/20; suite 441/441 (30 files, zero regressions); tsc clean. Both fixes were measured on operator-authorized probes before any code, ratified as designed, then landed.
- **PEL-026..PEL-041:** about 90 lesson candidates carried across 33 handoffs, consolidated into 16 topic entries; README index completed.

## Candidate progress
PC-008 HALTED (D-094), PC-007 HALTED (D-087), PC-006 HALTED (D-072), PC-005 PARKED 2/6 (D-083), PC-004 PARKED (D-058), PC-003 PARKED (D-034). Alternate terrain on file: RMP ammonia refrigeration.

## Readiness read
Engine capability is not the constraint. The admission gate's buyer and price inputs are now honest against the two contamination shapes seen in production, and the diag-before-run instrument has predicted six production runs exactly. The open limit is measurement, not machinery: categorizer changes still cannot be replayed over the run corpus, because exported reports keep summaries but not delivered text. As at SSP-058, Calla is blocked on terrain value. The next useful step is a terrain that survives a stage-1 price-ceiling check before any run.

## Next
SSP-060 opens on the 11-item checklist (suite baseline 441), then operator picks:
- RMP ammonia refrigeration with the operator's domain read and a price-ceiling check first
- the extractor fidelity arc
- the price-tables branch decision
- scout v2.5
- revivals

Production confirmation of D-095..D-097 waits for the next authorized gated run.

**End state:** gate CLOSED (9f46dce2c81a04ab8a8a39b4692fb440), PM2 stopped, budget 0/4 UTC 10-28, engine on main = origin/main with a clean tree (suite 441), scout repo d735402 unchanged, scratch cleared. Zero gated runs; D-096 and D-097 landed; PEL-026..041 written.
