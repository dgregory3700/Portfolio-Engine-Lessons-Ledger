# PEL-029 — Preserve Every Source URL at Read Time; Recover From Exports, Never Reconstruct

**Date:** 2026-10-28 · **Project:** Calla (goose01) · **Consolidates:** SSP-030, -031, -039, -057 candidates
**Tags:** _record keeping · URLs · probes · diag reads_

## Quick Recognition Test
A later session needs a source the record mentions by name only, or a re-typed URL 404s.

## Root Cause
Sessions logged the disposition of a source ("dropped", "read") but not its exact URL. Twice the URL had to be re-found by search (Illinois NetDMR guide, NCC letter). Once a reconstructed path 404'd while the recorded URL was live (erplan).

## What NOT to Waste Time On
Re-deriving a URL from the site structure or from memory.

## Canonical Fix Pattern
- Log the exact URL of every source at probe or read time, including dropped and diag-only sources.
- To reuse a source, copy its URL from the exported run report (reviewed-runs/) or the D-entry, never from a paraphrase.

## When to Apply This Again
Every probe, desk read and diag read.
