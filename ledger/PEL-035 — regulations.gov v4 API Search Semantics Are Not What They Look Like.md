# PEL-035 — regulations.gov v4 API Search Semantics Are Not What They Look Like

**Date:** 2026-10-28 · **Project:** Calla pain-evidence hunts · **Source:** SSP-052, -056 candidates
**Tags:** _regulations.gov · API · search · comments · rate limits_

## Quick Recognition Test
A comment search returns a count orders of magnitude off, or results cannot be tied back to a docket.

## Root Cause
Measured behaviour of `filter[searchTerm]`:
- It matches comment body text.
- Unquoted multi-word terms match ANY word ("spend hours" 340,047 hits vs 625 quoted).
- The index stems (frustrat == frustrating), and `OR` is not boolean.
- Results carry no docketId, but comment IDs are `<docketId>-<seq>`.
- `DEMO_KEY` rate-limits within a few calls.

## What NOT to Waste Time On
Boolean query construction, or using DEMO_KEY for anything beyond one call.

## Canonical Fix Pattern
- Quote multi-word phrases.
- Derive the docket from the comment-ID prefix.
- Use `include=attachments` on comment detail to get `fileFormats` URLs.
- Use the box key (`REGS_GOV_API_KEY`).
- The attachment downloads themselves sit behind the downloads.regulations.gov block (PEL-026).

## When to Apply This Again
Any regulations.gov comment-corpus query.
