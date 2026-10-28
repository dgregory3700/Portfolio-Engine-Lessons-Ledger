# PEL-037 — Terrain Screening; Run the Cheapest Kill First and Expect Value to Attract Incumbents

**Date:** 2026-10-28 · **Project:** Calla terrain selection (PC-005 through PC-008) · **Consolidates:** SSP-042, -043, -047, -054, -056, -057, -058 candidates
**Tags:** _terrain selection · incumbents · stage zero · screening order_

## Quick Recognition Test
You are choosing or desk-verifying a new candidate terrain.

## Root Cause
The measured regularities:
- The incumbent check is the cheapest and most frequent kill (FHA condo, WPS, organic, egg — all killed by desk read, zero runs).
- A burden floor works as a private-buyer filter. Dropping it surfaces government, survey and military collections, not thin-incumbent business terrain.
- Freshness is not a moat; purpose-built incumbents ride new-rule waves.
- Salience tension: high-voice terrains attract incumbents quickly.
- Inventory proxies cannot see incumbents. Per-respondent burden and cost measure market value, and value is what draws vendors. A thin-incumbent detector from the ICR inventory alone measured negative (n=13).

## What NOT to Waste Time On
- Building inventory-only incumbent detectors.
- Re-desk-reading terrain already set aside.

## Canonical Fix Pattern
- Generate candidates from the ICR-inventory scout (stage zero, inadmissible by construction), not from memory.
- Grep the D-ledger and handoffs for the OMB number and terrain name before any stage-1 read.
- Run the incumbent check first, including the class-3 test: does the buyer class run a vertical ERP that already carries the compliance feature?
- Calibrate any new stage-zero instrument against halted candidates with known outcomes before reading its ranking.

## When to Apply This Again
Every terrain hunt.
