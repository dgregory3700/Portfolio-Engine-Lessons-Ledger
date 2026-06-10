# PEL-020 — Dead-Var Verdicts Must Be Derived by Grepping the Surface That CONSUMES the Var, Not the Surface It Points At

**Date logged:** June 10, 2026
**Project:** HarmonyDesk Pro — Cat 3F-1 (Vercel dead-environment-variable cleanup; deck-clear before Cat 2G)
**Related entries:** PEL-012 (verify, don't trust prior assertions). Closes the last open Cat 3F item.

---

## Symptom / context

The Cat 3F session (June 4) gathered intel marking `EC2_INTERNAL_URL` as "confirmed dead via EC2 grep — safe to delete from Vercel." During Cat 3F-1 execution (June 10), a fresh full derivation of the in-use set proved that verdict **wrong**: `EC2_INTERNAL_URL` is live in `src/app/api/sessions/route.ts` (dashboard repo), where the POST handler fires the session-confirmed email at `${process.env.EC2_INTERNAL_URL}/internal/email/session-confirmed` with **no fallback**. Deleting it would have silently broken confirmation emails for manually-created confirmed sessions — silent because the fetch is wrapped in a fire-and-forget try/catch that only logs.

## Root cause / mechanism

- `EC2_INTERNAL_URL` is a **Vercel** env var whose *value* points at EC2. The original "dead" verdict came from grepping the **EC2** codebase — the surface the var points at, not the surface that reads it. EC2 never reads its own URL from a Vercel var, so the grep was guaranteed to come back empty regardless of whether the var was alive.
- The category error: a var's home is where `process.env.X` is evaluated, not where its value resolves to. URL-shaped vars are especially prone to this inversion because their value names the *other* machine.
- Compounding factor: GitHub's `search_code` returns matching files but not always matched lines, and is unreliable on compound underscore identifiers. The only authoritative derivation is reading every file that contains `process.env` in full.

## Diagnostic technique / verification

Canonical 3F-1 derivation procedure (reusable for any Vercel/EC2 split project):
1. **Dashboard repo:** `search_code` for the literal `process.env` (reliable — it's a common literal, not a compound identifier) to enumerate files, then `get_file_contents` on **every** file and extract `process.env.X` references by reading, not by trusting search snippets. (HD Pro: 30 files.)
2. **EC2:** one SSH grep: `grep -rhoE "process\.env\.[A-Z0-9_]+" server.js routes/ services/ | sort -u`, plus ecosystem env-block identifiers and `.env` key names (names only — never print values).
3. Diff the union against the live Vercel var list (operator reads UI; screenshot truncates long names — **confirm exact full names in the UI before any delete**).
4. Delete only vars with zero references on their consuming surface; redeploy; smoke test.

## Outcome (June 10, 2026)

- **Deleted (3):** `STRIPE_WEBHOOK_SECRET` (zero dashboard refs — webhook verification lives only on EC2; the Vercel copy still held the breach-flagged April 1 value, hence Vercel's "Needs Attention" badge — deleting it removed a leaked secret's last resting place), `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` (zero refs), `HD_PRO_PRICE_ID` (zero refs — the price ID is hardcoded as a const in `stripe/checkout/route.ts`).
- **Kept despite prior delete-intel:** `EC2_INTERNAL_URL` (live, see above), `NEXT_PUBLIC_API_URL` (live in `billing.ts` + `topbar.tsx` billing-status calls).
- **Self-resolved:** `NEXT_PUBLIC_EC2_URL` was already absent from Vercel — the expected rename leftover had been deleted in a prior session.
- Redeploy green; login + /calendar + public /request/[slug] smoke passed.

## What NOT to waste time on

- Re-deriving on memory or on a prior session's intel — two rotations had touched the Vercel env between the June 4 intel and execution. Intel expires; derivation doesn't.
- Treating a fallback (`process.env.X || "default"`) as license to delete X — the explicit var documents the dependency; keep it unless there's a positive reason to remove.
- Deleting a var whose displayed name is truncated in the Vercel UI without clicking in to confirm the full name first.

## When to apply again

- Every future env hygiene pass on any DuelTech project with a Vercel/EC2 (or any multi-surface) split.
- Any time a prior session's "safe to delete" claim is older than the last change to that env surface.
