# PEL-034 — reginfo and OMB Data Surfaces; Use the Bulk XML and Supporting Statements, Not Scraped Pages

**Date:** 2026-10-28 · **Project:** dueltech-icr-scout and Calla terrain work · **Consolidates:** SSP-041, -042, -043, -046, -047, -052, -054 candidates
**Tags:** _reginfo · PRAXML · ICR inventory · Supporting Statements · docx_

## Quick Recognition Test
You need burden hours, responses, respondents or per-IC breakdowns for federal information collections.

## Root Cause
reginfo's HTML pages are lossy (garbled IC titles on EPA UST pages; nested-div truncation on PRAViewICR), while the structured data exists elsewhere:
- reginfo publishes bulk XML at `PRAXML?type={inventory|expiration|concluded}`. Inventory is ~105 MB daily, with burden, respondents, affected public, ObligationCode, IC titles and per-IC figures all structured.
- Schema asymmetry: `BurdenHour` carries a total first, while `NumberResponses` is per-IC with no total. Sum the responses and take the first burden value.
- Supporting Statements are docx-first.

## What NOT to Waste Time On
- Paging PRAViewICR per `ref_nbr` for inventory-wide questions.
- The omb.report mirror (Cloudflare-blocked).
- Treating a ~10 s connect-block from the box IP as an outage; retry, and check from another vantage before falling back to a cache.

## Canonical Fix Pattern
- Use the inventory XML for any cross-ICR or per-IC decomposition; a box-side script settles Prong 1 in a minute.
- For single-IC ICRs, read the Supporting Statement (Q12 burden table) through reginfo `DownloadDocument`.
- Read .docx with stdlib zipfile plus a regex tag-strip, splitting on `w:p` and taking `w:t` contents; no python-docx needed.

## When to Apply This Again
Every terrain hunt and every V1 burden question.
