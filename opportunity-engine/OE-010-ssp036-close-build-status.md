# OE-010 — Calla build status at SSP-036 close (2026-08-08 UTC)

Engine HEAD at close: 31f9e37 (goose01). Sessions SSP-030 through SSP-036 have shipped D-038 through D-050.

## Where Calla stands relative to "ready"

**Evidence-surfacing capability is now broad.** This session added the last two structural pieces that were on the books: the WR proximity matcher (D-049) closed the third vocabulary-trailing list family with a structural fix rather than more phrase enumeration, and the OCR path (D-038) ran in production for the first time (D-050), surfacing engine-admissible evidence from a scanned public record at 94.7% mean confidence. The engine can now read: HTML, Flate PDFs, windowed slices of large PDFs, JSON API responses (regulations.gov, with declared-identity categorization), and scanned documents via opt-in OCR with mandatory provenance disclosure.

**The instrumented-record layer is confirmed.** D-047 annex and D-049 both production-confirmed this session. D-048 (substantive-anchor categorizer) remains the one unconfirmed change — it needs a .gov burden-doc slate.

**The known structural weakness has a name and a measured basis.** Selection competition is at seven recorded sightings, and D-050 demonstrated a new surface: the annex's own per-family slot can be occupied by dense unrelated content. The refinement (cap or tie-break) is queued with a clean two-run measured basis and is record-only, so it carries no detector/gate risk.

## Candidate state
- PC-004 (EPCRA Tier II/TRI): UNCONDITIONALLY ADMITTED. Validation stage not yet begun — WTP, data availability, work-removal all open. This is the main line of advance.
- PC-003 (NPDES DMR prep): PARKED, revival-eligible; record strengthened this session (first engine-surfaced OCR evidence admitted; approval motion honestly still operator-verified-only). NPDES ICR comment window closes 09-01 — the docket watch may open a fresh pain vein before then.
- PC-001/PC-002: closed.

## What still needs doing (queue at handoff)
1. Annex capture refinement (fresh measured basis)
2. PC-004 validation stage (the substantive arc; a .gov burden slate there also confirms D-048)
3. FR-PDF extractor defect (blocks the Federal Register evidence class)
4. package-lock tracked-or-not call

## Honest assessment
Calla's machinery is closer to "ready" than its pipeline: the engine now reads nearly every evidence class it has encountered, with disciplined provenance, but only one candidate is admitted and none has completed validation. The bottleneck is deliberate — validation rigor, not capability — and the next sessions should spend budget on PC-004 validation rather than further engine arcs unless a validation run surfaces a new defect.

Suite at close: 209/209. Two gated runs this session, both exported and admitted (D-050). Gate closed and verified; budget 2/4 UTC 08-08.
