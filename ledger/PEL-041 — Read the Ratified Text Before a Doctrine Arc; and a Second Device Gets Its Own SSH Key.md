# PEL-041 — Read the Ratified Text Before a Doctrine Arc; and a Second Device Gets Its Own SSH Key

**Date:** 2026-10-28 · **Project:** Calla governance and box access · **Source:** SSP-058, -059 candidates
**Tags:** _doctrine drift · ledger reading · SSH keys · multi-device_

## Quick Recognition Test
(a) A doctrine arc is framed as "stop doing X". (b) A new laptop needs access to the box.

## Root Cause
(a) D-039 Prong 3 already said it "does not veto". The arc to stop vetoing was really a correction of practice drift, carried by summary lines in later entries (D-067 context) that restated the rule wrongly. (b) Copying a private key between machines spreads one credential across devices and loses per-device revocation.

## What NOT to Waste Time On
(a) Drafting new doctrine against a paraphrase. (b) Moving `.pem` or ed25519 private keys between laptops.

## Canonical Fix Pattern
(a) Read the ratified entry text itself before designing any doctrine change. If practice drifted, record a correction rather than new doctrine.
(b) Generate a fresh key on the new device. Authorize it into `authorized_keys` from the device that already has access. Keep inbound 22 pinned to the home /32, adding a temporary /32 only while away and removing it on return.

## When to Apply This Again
(a) Every doctrine arc. (b) Every new device.
