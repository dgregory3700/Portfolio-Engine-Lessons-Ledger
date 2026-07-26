# OE-008 — SSP-029 close build status (2026-07-26)

**Session span:** 2026-07-24 → 2026-07-26 UTC (two UTC days; one run budget touched on each: 1/4 and 1/4).

## What shipped
- **D-035 LANDED** (goose01 `b797eeb`): per-URL source char windows (`OE_SOURCE_CHAR_WINDOWS`, `<url>=<start>:<end>`) with window-then-cap slicing, plus truncation transparency — every evidence item and run report now discloses totalExtractedChars / deliveredChars / window, with a head-truncated flag. Backward compatible (no window = legacy head slice). Suite 115/115 including new permanent harness `scripts/window_test_run15.ts` (17/17). Summary selection competition explicitly out of scope.
- **D-036** (`c2b3eb1`): Camrosa WIMS packet admitted via windowed re-fetch (`OE-PREVIEW-20260724T141417Z`) — Table 1 license fees (Total One-Time $20,298) + Table 3 Annual SMA $3,653.64/yr. PC-003 revival record: V4 second stated annual renewal term; V6 incumbent price floor n=2→n=3.
- **D-037** (`0c3441d`): Illinois EPA NetDMR Quick Answer Guide admitted via windowed re-fetch (`OE-PREVIEW-20260726T124105Z`), quality A official documentation — DMR import file types (tst/csv/zip) + CSV format spec. **D-025 federal-import caveat closed on the acceptance branch.**

## Honest state
The first designed engine-capability arc went design → ratify → land → proved itself in production twice in one session, converting two long-blocked evidence classes into admitted record. PC-003 remains **PARKED (revival-eligible)** — the revival record is materially stronger (V2 both branches, V4 two terms, V6 three price points) but V5 still gates revival and was untouched. Remaining engine gaps: OCR path (scanned public records) and the selection-competition pattern (3 measured instances; windowing mitigates targeted slates only). Throughput stays session-bound by design (worker stopped, discovery operator-driven).

## Next
Docket re-poll EPA-HQ-OW-2008-0719 DUE at SSP-030 open (~07-27). Queue (operator picks): OCR arc, selection-competition, PC-004 intake. Handoff: `handoffs/SSP-goose01-030-post-d035-queue.md` (close commit `cf1f2eb`).

**End state:** gate CLOSED (verified), PM2 stopped, tree clean (package-lock.json exception), all ledgers pushed.
