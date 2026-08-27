# OE-018 — build status at SSP-046 close (2026-08-27 UTC)

**Where Calla stands:** engine mature and stable; the active frontier is **candidate terrain supply**, not engine capability. Zero engine code this session (suite 277). All SSP-046 work was stage-zero/stage-1 scouting (inadmissible, outside the evidence boundary, no D-entries).

## Shipped this session
- **Scout v2.2** (dueltech-icr-scout, 73a8eaf): doc-vs-physical title signal; retired the hrs/resp depth reward for log10 response-breadth; extended institutional clusters (+IRS/26CFR, +CBP/19CFR, +FinCEN/31CFR); D1 no auto-interposition. Parser correctness fix: reginfo NumberResponses is per-IC with no grand-total block → responses summed across ICs (verified 2070-0195 = 27,321,155, was 12,087 IC1-only); BurdenHour stays first-grab (grand-total-first).
- **Scout v2.2.1** (0f19738): breadth ceiling log10(min(resp,5M)) + transaction penalty (hrs/resp<0.05 → 0.35×) to demote per-transaction mega-collections; physical penalty extended to per-unit product families (textile/fiber/apparel/garment). Fresh 20260827 slice committed (b5e6af8), 980 survivors.

## Candidate progress
- **PC-006 terrain: still UNSELECTED.** Killed this session at Prong-3 (all inadmissible, no D-entry): FHA condo 2502-0610 (tech incumbents CondoTek/HomeWiseDocs + lender-concentrated buyer), WPS 2070-0190 (free EPA/PERC/extension instruments + trivial roster doc slice), organic 0581-0321 (purpose-built incumbent Quick Organics + free FarmOS + certifier portals + USDA INTEGRITY), egg 0910-0660 (~4,100-farm concentrated TAM + free FDA/extension + crowded food-safety-plan SaaS).
- Carried: PC-003 PARKED (revival-eligible; NPDES renewal-cycle lead to verify), PC-004 PARKED, PC-005 halted at admission PARK 3/5 (D-066).

## Readiness read
The engine is "ready" and the scout is now well-tuned (v2.2.1). The binding constraint is the **thesis**: across PC-004/005/006, ICR terrains with enough burden+WTP to matter attract incumbents, and small/low-income buyers get an agency-subsidized free tool by design. Lead RRP is the sole Prong-3 survivor and reached only PARK 3/5. Floor-drop exploration confirmed the ≥100K burden floor is effectively a private-buyer filter — lowering it surfaces government/survey collections, not thin-incumbent business terrain. **Thesis-review deferred to SSP-047** (operator banked for the day).

## Next
Operator decision at SSP-047: continue the ICR-terrain hunt (fragile top-25 residue) vs. step back to generate PC-006 on a different axis. Optional scout v2.3 (DOT/FAA/EPA-air clusters) if the hunt continues.

**End state:** gate CLOSED (32c1a811), PM2 stopped, budget 0/4 UTC 08-27, engine + scout repos tree-clean, scratch cleared. Zero gated runs, zero engine D-entries (next D-067).
