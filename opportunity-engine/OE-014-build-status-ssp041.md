# OE-014 — Calla build-status at SSP-041 close (2026-08-13 UTC)

**Where Calla stands relative to "ready":** the engine is feature-mature for the four terrains exercised so far (PC-001 through PC-004, all PARKED). SSP-041 was a backlog-clearing session — no candidate advanced, no gated runs. The engine gained one detector-selection improvement (D-061) and closed a housekeeping item (D-062). No active candidate; PC-005 terrain selection is the next substantive arc.

## This session (SSP-041)
- **D-061 — PRICE_LABEL_TERMS structural amendment (SHIPPED):** first vocabulary-trailing family to clear the D-048 count bar (2 instances / 2 terrains: Cartersville PC-003 "purchase ... for $27,140.00", RMA PC-004 "Typical Range: $1,000-$3,000 per facility"). New exported `hasPriceLabel()` joins two structural regexes (per-unit `$ ... per <unit>`; purchase-predicate `purchase/acquire/procure ... for/of/is/at/: $`) to the lexical list, wired into all 4 price call sites, all under the anchorBonusOn guard (D-007 suppression byte-identical). Measured-minimal per D-048: pay/cost/buy excluded (the FP surface in audit narrative). Permanent harness run27 23/23; all 5 real run11 audit-narrative dollar passages stay OFF; run11 18/18 discrimination intact. Code badfe85 + ledger b810c69.
- **D-062 — package-lock.json kept tracked (RATIFIED):** valid lockfileVersion-3 file, entered accidentally at 1f9314d (D-045) but belongs in the repo (npm default, reproducible clean-clone/AMI installs per D-038). No repo-tree action; ledger entry only. Ledger 5af04ee. Backlog item closed.
- **SS-A docx desk read (NOTE, non-D):** EPA ICR 1352.17 SS-A read box-side (stdlib zip+regex, engine still has no docx path). Confirmed Tier II figures (462,640 facilities / 6.6M hrs / $311M total public burden) and newly quantified per-facility unit burden (Exhibit 1: mfr Small 13.89 hrs / Medium 78.24 / Large 117.24). Does NOT retire D-052 caveat 1 (path, not existence). Located the reginfo-HTML admissible-path lead for PC-004 revival trigger 3. Note 03c118c.

## Portfolio state
- PC-001 PARKED (V1 fail, closed) / PC-002 PARKED (V1 fail, closed) / PC-003 PARKED (D-034, strong V-record, structural V5 fail) / PC-004 PARKED (D-058, strongest parked profile, evidence-availability fails with named revival triggers)
- No active candidate. PC-005 terrain selection = next arc (D-039 three-prong pre-screen doctrine).

## Pending production confirmations (ride next authorized gated run)
D-048 exclusion arm, D-059 (OCR re-split), D-060 (FR-PDF/HTML extractor), D-061 (price structural predicates). All are capability-shipped and harness-verified; each needs a live run carrying the relevant source type to confirm in production.

## What still needs doing to reach "ready"
Calla remains a discovery/validation engine, not a shipped product. The path to a built product runs through a candidate that clears the full ladder AND has a revenue mechanism that survives V5 (the gap that parked both PC-003 and PC-004). PC-005 terrain selection is where that search resumes.

## End state
Gate CLOSED, PM2 stopped + dump stopped, budget 0/4 UTC 08-13, /tmp clean (system entries only), repo tree clean. HEAD goose01 = 35c7f0b (SSP-042 handoff). Suite harness-summed 277, tsc clean. Zero gated runs, zero probes this session.
