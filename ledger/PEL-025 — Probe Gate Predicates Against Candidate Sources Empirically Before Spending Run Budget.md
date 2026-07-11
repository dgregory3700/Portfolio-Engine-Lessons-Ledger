# PEL-025 — Probe Gate Predicates Against Candidate Sources Empirically Before Spending Run Budget; Plausible-Looking Sources Fail on Marker Counts

**Date:** 2026-07-11
**Project:** Calla / DuelTech Opportunity Engine (goose01)
**Session:** SSP-023 (PC-003 pain-criterion arc)
**Severity:** Process lesson — prevented wasted gated-run budget; generalizes to any budget-limited evaluation pipeline

## What happened

SSP-023 needed a source that would close the "direct operator pain is evidenced" admission criterion for PC-003 (NPDES DMR preparation). Two sources looked strongly viable on inspection:

1. **EPA Compliance Advisory on DMR submission** (epa.gov static PDF) — .gov host, quantified problem statement ("DMR reporting violations account for over half of the facilities in SNC"), dense error/mistake language. By eyeball, a plausible pain source.
2. **GAO-21-290** (NPDES compliance data quality) — guaranteed findings voice ("we found", "we recommend"), .gov, burden vocabulary.

Instead of slating them into a gated run (budget: 4/day, operator-authorized), we wrote a ~40-line repo-resident probe (`scripts/vet_023_pain_probe.ts`) that runs the engine's own extractor on a locally-curled copy and counts hits against the exact anchor lists read from the gate code (`OPERATOR_PAIN_ANCHORS`, `OFFICIAL_BURDEN_ANCHORS`, `FINDINGS_VOICE_MARKERS`, `PROCEDURAL_DOC_MARKERS`), then evaluates the routing predicate literally.

**Both plausible sources failed empirically:**
- EPA advisory: 1 burden anchor, 0 findings voice, 0 pain anchors — tips-shaped, not findings-shaped. Would have routed official_documentation (not pain-eligible) and contributed nothing.
- GAO-21-290: the 2.4MB PDF crashed the engine extractor outright (RangeError, max call stack, in the Tj/TJ operand regex) — it would have failed identically inside a live run; the HTML fallback host 403'd the engine UA.

Meanwhile a source that looked *risky* on inspection (tpomag.com trade article behind an apparent login wall) probed clean — the wall was client-side, the body was server-side, and a second adapter-level probe (`scripts/vet_023_tpo_probe.ts`, route + assembled evidenceSummary, no DB writes) confirmed it routed `operator_pain` with an anchor in the summary. Runs 3 and 4 then passed on the first attempt each.

## The lesson

**Eyeball plausibility of a source and mechanical pass/fail of a scoring pipeline are uncorrelated enough that every slate candidate should be probed against the real predicates before consuming budget.** Three specifics:

1. **Read the anchor lists from the code, not from memory or docs**, at probe time. The probe duplicated the lists verbatim from the adapter source read minutes earlier; a handoff-doc paraphrase of "burden/tedious/etc." would have missed exact-match subtleties (e.g., "time-consuming" vs "time consuming" are separate entries; "frustrat" is a stem).
2. **Run the engine's own extractor, not a generic one.** The GAO PDF failure was an engine-extractor hard-fail invisible to `pdftotext`-style tools. Probing with anything other than the production code path gives false confidence.
3. **Probe the assembly layer too when the criterion is scored downstream of extraction.** Marker presence in full text ≠ marker presence in the 900-char evidenceSummary the gate actually scans. The adapter-level probe (discover() on a single URL, evidence-inadmissible, no durable writes) is cheap and answers the real question.

## Cost/benefit measured

Probe cost: ~40 lines, two commits, minutes. Benefit: two doomed sources excluded and one non-obvious source validated before run 3; runs 3 and 4 both ADMIT 5/5 first-try. Without probing, the plausible-but-doomed slate would likely have burned 1–2 of the day's 4 runs on PARK 4/5 repeats.

## Reusable artifacts

- `scripts/vet_023_pain_probe.ts` — marker-count any local PDF/text file against gate anchor lists + predicate evaluation.
- `scripts/vet_023_tpo_probe.ts` — adapter route + evidenceSummary probe for a single URL (arg), evidence-inadmissible.

Both are additive scripts; no engine code changed. Pattern carried into the SSP-024 handoff as a standing vet-rule extension: marker-count candidates before slating.
