# OE-021 -- build status at SSP-049 close (2026-09-10/11 UTC)

**Where Calla stands:** the biggest engine session since the OCR arc -- four ledger entries, three code landings, suite 277 -> 328, zero gated runs. The headline is doctrinal: the admission gate could not, as coded, ever pass the buyer/procurement criterion for the private-buyer terrain D-068 now mandates. That contradiction is fixed (D-073), and two engine defects it uncovered on the way (numeric-entity decoding, storefront categorization) are fixed with it. PC-005 is unblocked with a diag-verified run package expected to land PARK 4/5.

## Shipped this session
- **D-072 landed** (679bb4a): PC-006 HALTED at admission PARK 2/5. Operator chose to accept the PARK rather than spend runs on a third-angle hunt. Revival-eligible; lead recorded.
- **Take-stock direction 1 executed: revive PC-005.** Claude-side scouting found a priced per-job market for the D-064 wedge (contractorsorganization.org 1-Job RRP Compliance & Record Keeping Kit $49.99, 3-Job $99.99, per-form pads $21.95; zotapro.com organizer $75). All three pages probed PASS under engine UA.
- **D-073 landed** (b48f513 / ledger 7e466f2): private-buyer WTP route for the admission-gate buyer criterion. Every pre-existing route into `buyer_evidence` was public-buyer shaped (RFP/award, public-record procurement, funded mandate) -- structurally incompatible with the D-068 pre-screen. The route applies the D-027 named-deliverable standard and D-054 R1 / D-057 Reading A precedent at admission: a `pricing_budget`/`competitor_vendor` item whose citable summary carries a price and a seed term in one sentence unit, with >=3 seed terms overall. Route disclosed in trace, notes and report. Harness run28 20/20; consequence replay over 60 reports found 2 criterion-level flips, both on pricing artifacts the ledger had already ruled WTP-relevant, no ruled verdict changed.
- **D-074 landed** (26a5598 / bd41409): `decodeEntities` ignored numeric character references; WooCommerce emits every "$" as `&#36;`, so ACO's prices were invisible to every price detector in the engine. 20 of 60 reviewed-run reports carried undecoded references. Single-pass decoder + post-decode whitespace collapse (a second-order defect the harness found). Run29 24/24; live re-probe shows 19 price artifacts, 0 entities left.
- **D-075 landed** (a103d8a / c0522d4): diag-before-run showed all three vendor pages classifying `operator_pain`, and the kit page's marketing copy passing the gate's pain predicate -- vendor copy as pain, the D-005/D-013 hole on a page shape (physical-goods storefront) the corpus had never met. Storefront clause: price + existing storefront-action term ("add to cart"/"buy now"/"order now") -> commercial, pain-ineligible. Zota not covered (only "quantity"/"list price" -- single-terrain vocabulary, below the D-048 bar), dropped from the slate, pinned as honest non-coverage. Run30 7/7.
- **7-URL PC-005 slate diag-replayed through the patched pipeline and the real gate: PARK 4/5** -- buyer criterion via the WTP route, direct pain via McCadden, only the two by-design prongs failing. Expectation, not a production result.

## Candidate progress
- **PC-005: revival unblocked.** Run package ready in the SSP-050 handoff (ratified 5 URLs + two ACO pages, cap 7, 25s timeout, case-page window re-verified pre-run because D-074 changed extraction). Expected 4/5 = the highest admission standing any private-buyer terrain can reach until the Mirage/Relocation prongs are addressed at V-stage.
- **PC-006: HALTED** at admission PARK 2/5 (D-072), revival-eligible.
- PC-003 PARKED (D-034, D-068 terms). PC-004 PARKED (D-058).
- The take-stock question (is the reginfo ICR inventory itself producing thin-buyer candidates?) is partly answered: at least one wall was the gate's, not the terrain's.

## Readiness read
Diag-before-run has grown from "replay the scorer" to "replay categorizer -> summary -> criterion predicates -> gate" and it paid for itself three times in one session with zero budget spent: a route that would never have fired, a price that was invisible, and a vendor page that would have been admitted as pain. The engine is more honest tonight than it was this morning, and the next run is the first production confirmation of all three changes at once -- worth reading the report line by line.

## Next
SSP-050 opens on the standard 7-item checklist, then operator go/no-go on the PC-005 revival run (D-076). Queue behind it: Zota storefront coverage (needs a second terrain), PC-006 third-angle hunt, take-stock direction 2, scout v2.3.

**End state:** gate CLOSED (9f46dce2c81a04ab8a8a39b4692fb440), PM2 stopped, budget 0/4 UTC 09-11, engine tree clean at 134600d = origin/main, scratch cleared. Zero gated runs, zero void runs, three authorized probes.
