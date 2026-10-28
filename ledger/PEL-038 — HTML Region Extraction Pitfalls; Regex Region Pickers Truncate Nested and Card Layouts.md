# PEL-038 — HTML Region Extraction Pitfalls; Regex Region Pickers Truncate Nested and Card Layouts

**Date:** 2026-10-28 · **Project:** Calla engine extractor (`extractPreferredHtmlRegion`) · **Consolidates:** SSP-045, -048, -053, -059 candidates
**Tags:** _HTML extraction · region picker · regex · vendor pages_

## Quick Recognition Test
A 200 page with real static HTML extracts a fraction of its text, or single-digit characters, often at quality E.

## Root Cause
Regex cannot balance tags:
- A non-greedy `role="main"` match stops at the first closing tag of the same name, so nested structure truncates the region (reginfo PRAViewICR: 3,239 of 104,137 chars).
- Card-layout landing pages put one `<article>` per card, and a first-match picker keeps one card (fixed by D-082).
- A region selector page can deliver almost nothing (lermansenter.com: 7 chars).
- Related category trap: first-party vendor copy hosted on a marketplace domain (App Store) escapes the vendor-page guard.

## What NOT to Waste Time On
Tuning windows or seeds on a source whose region extraction already failed.

## Canonical Fix Pattern
- Check `totalExtractedChars` against page size in diag for every HTML source.
- On a large gap, dump the extracted text and compare it to a full tag-strip.
- Treat each new layout shape as a measured extractor defect with its own arc.

## When to Apply This Again
Every new HTML host, especially vendor landing pages and government portals.
