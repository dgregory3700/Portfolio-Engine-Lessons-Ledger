# OE-012 — Calla build status at SSP-038 close (2026-08-11 UTC)

## Where Calla stands relative to "ready"
- PC-004 (EPCRA Tier II / TRI facility chemical reporting) is the live candidate and the furthest any candidate has progressed: unconditionally ADMITTED (D-043/D-046) and now V1 PASS (D-052) + V2 PASS (D-053) on the D-052 validation ladder. V3+ (willingness-to-pay, Tier A/B) is next, with interpretation rulings pre-ratified (D-054 R1-R4, including the B+B two-vendor floor).
- PC-003 (NPDES DMR prep) remains PARKED and revival-eligible per D-034; the NPDES ICR comment window (closes 2026-09-01) is a dated watch item that could freshen its pain record.
- Engine instrumentation matured this session: D-049 proximity matcher earned its FIRST production hits, D-051 annex ranking confirmed in production. D-048 burden-categorizer confirmation still pending (needs a burden-doc slate).

## What still needs doing before any build decision
1. PC-004 V3 WTP: second independent vendor with published per-facility Tier II pricing (hunt), then the V3 run package (RMA + second vendor). V4-V6 follow the ladder; V6 carries the declared wedge risk (funded incumbents in multi-facility assembly).
2. Known engine debt, none blocking V3: FR-PDF/encoding extractor family (now three instances — FR mojibake, Form 3320-1, NY DHSES 1,275 vs 12,889 chars), OCR passage segmentation (D-051 Ruling C), package-lock tracking call.
3. No build is authorized: doctrine requires the full V-ladder plus an explicit operator build ratification, which PC-004 has not reached.

## Session discipline notes
One gated run (1/4 budget UTC 08-11), exported and admitted same-session under D-053; gate atomic open/restore md5-verified via a detached trap-guarded runner (new pattern worth keeping); scratch fully cleared; tree clean; HEAD ce19414.
