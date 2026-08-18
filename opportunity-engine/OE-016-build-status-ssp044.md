# OE-016 — Build status at SSP-044 close (2026-08-16)

## Where Calla stands relative to "ready"

Calla remains a working, disciplined opportunity engine. No engine code changed this session — SSP-044 was a pure candidate-arc session (PC-005 taken to admission verdict and halted). Suite stands at 277 (harness-summed) from SSP-043; gate discipline, detached trap-guarded runner, and export/ledger flow all exercised cleanly across three gated runs.

## This session's engine-relevant events

- Three gated runs executed against the same 5-URL PC-005 slate. Two operator/Claude-surfaced operational issues, both diagnosed and worked around without code change:
  - **OE_MAX_SOURCE_URLS default 3** silently truncated a 5-URL slate to its first 3 (run 1 void). Worked around by setting the env var to slate length. Candidate engine-UX fix: emit a log line when the slate exceeds the cap.
  - **OE_FETCH_TIMEOUT_MS default 10,000 ms** aborted a 2.34 MB PDF that takes ~18.9 s to fetch (run 2, quality E). Worked around by raising to 25,000 ms (run 3 recovered it, quality A). Candidate: size- or host-aware timeout, or a probe-time check.
- **extractPreferredHtmlRegion region truncation** (queued defect): the role="main" non-greedy first-close-tag match collapses reginfo PRAViewICR to 3,239 of 104,137 chars. Regex cannot balance tags; a real fix needs a depth-aware region selector. Not blocking — the IC-level PRAViewIC page carried the needed burden figure.

## What still needs doing (unchanged from SSP-043 unless noted)

- Engine defects queued: HTML region truncation (new this session), OCR passage segmentation sibling classes (GAO TOC colon-glue), FR-PDF encoding edges beyond D-060's coverage.
- Pending production confirmations still riding a future gated run: D-048 exclusion arm, D-059, D-060, D-061 — note the PC-005 runs did NOT exercise these (no burden-exclusion predicate, no OCR/price-list, no per-unit/purchase price passage fired). They remain pending.
- The four-run-per-day budget, gate-closed-by-default, and manual allowlisted slates all held.

## Candidate ladder status

- PC-001 PARK, PC-002 PARK, PC-003 PARK (V1–V4/V6 pass, V5 park), PC-004 PARK (ladder complete, strongest parked profile), **PC-005 PARK 3/5 at admission — HALTED (D-066), weaker entry than PC-004, not pursued**.
- Next: **PC-006** terrain selection (SSP-045). No terrain selected yet.

## End state

Gate CLOSED (md5 32c1a811), PM2 worker stopped + dump stopped, budget 3/4 UTC 2026-08-16, both repos tree-clean, all scratch cleared. HEAD at close-adjacent 5753da9 (D-066) with SSP-045 handoff committed after.
