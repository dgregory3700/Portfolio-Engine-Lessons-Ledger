# PEL-018 — PM2 Non-Block Env Vars Refresh ONLY on a Full Recreate; `pm2 restart` and `pm2 reload <file>` Both Fail

**Date logged:** June 9, 2026
**Project:** HarmonyDesk Pro (EC2 backend, PM2 process `harmonydesk`, `/apps/harmonydesk/harmonydesk-mvp/`) — Cat 3 `SUPABASE_SERVICE_ROLE_KEY` migration
**Related entries:** **Corrects PEL-016** (its mechanism and conclusion are both wrong). Refines PEL-015 and PEL-017 (the block-var `reload <file>` doctrine stands; this entry only narrows what it does NOT cover).

---

## Symptom / context

Cat 3 migrated `SUPABASE_SERVICE_ROLE_KEY` (legacy JWT → `sb_secret_`). The carried-in plan (per SSP and PEL-016) was: non-block var → plain `pm2 restart`. Before swapping the real key, a non-destructive diagnostic compared each secret's value in `.env` against the running process's `pm2 env`:

- `SUPABASE_SERVICE_ROLE_KEY`: `.env` == `pm2 env` (matched — never changed since first start)
- `STRIPE_WEBHOOK_SECRET`: `.env` = `…5XSc9t` but `pm2 env` = `…LcRJbv` — **MISMATCH**

The mismatch is the recognition signal: a var "rotated" in `.env` (Cat 3G, 6/6) whose running process still carried the old value — directly contradicting PEL-016's claim that any respawn refreshes non-block vars.

## Root cause / mechanism

PEL-016 assumed non-block vars reach the process via `server.js`'s own `dotenv.config()` and therefore never touch PM2's cache. Wrong on two counts:

1. **`ecosystem.config.cjs` itself calls `dotenv.config({ path: __dirname/.env })` at config-parse time** (line 7). When PM2 first evaluates the ecosystem file (`pm2 start <file>`), this loads the *entire* `.env` into the env PM2 captures and forwards to the child. So **every** `.env` var — block or not — lands in PM2's injected env, and `pm2 env <id>` **does** show non-block vars. (PEL-016's "pm2 env won't show it" is false.)

2. **`server.js`'s own `dotenv.config()` (line 30) is a no-op for refresh.** The route imports at lines 22–28 transitively load the modules that read `process.env` (e.g. `services/supabase.js` reads the key at module load). In ESM those imports fully evaluate *before* line 30 runs; and `dotenv` never overwrites a key already present in `process.env`. The value the app uses comes entirely from PM2's injected env, set once at first `pm2 start`.

PM2 refresh semantics over that injected env:
- `pm2 restart <name>` — respawns from cached definition + cached env. No file re-read. Non-block var unchanged.
- `pm2 restart --update-env` — re-applies the current daemon/CLI env, not a `.env` re-read. Unchanged.
- `pm2 reload <file>` — re-evaluates the ecosystem file and re-applies only the declared `env:` block (block vars refresh — PEL-017). The captured ambient env carrying non-block vars is NOT re-captured. Unchanged.
- `pm2 delete <name> && pm2 start <file>` — fresh evaluation rebuilds the captured env from current `.env`. **All** vars refresh.

## Diagnostic technique / verification (6/9/26)

A harmless probe var settled it without guessing:
1. Appended `CAT3_PROBE=...` to `.env` (backup first).
2. `pm2 restart harmonydesk` → `pm2 env`: **ABSENT**.
3. `pm2 reload /apps/.../ecosystem.config.cjs` → `pm2 env`: **ABSENT**.
4. `pm2 delete harmonydesk && pm2 start /apps/.../ecosystem.config.cjs` → `pm2 env`: **PRESENT**.
5. Removed probe, recreated clean.

The recreate also resynced `STRIPE_WEBHOOK_SECRET` from `…LcRJbv` → `…5XSc9t`, confirming the mismatch was a stale process, not a stale file.

## Why PEL-016's "conclusive proof" was a false positive

PEL-016 cited a Cat 3G Stripe Resend returning 200/`signature_verified` as proof the new webhook secret was live. It was not. The process held the **old** secret throughout. The Resend passed because the old secret was inside its 24-hour delayed-expiry window — Stripe still signed with it. A dual-valid-secret window masks a stale process. After that window expired (~6/7), the process would 400 on real Stripe traffic until the 6/9 recreate moved it to the new value; with no live customers this went unnoticed.

## Canonical fix / pattern

- **Non-block var (NOT in `ecosystem.config.cjs` `apps[0].env`):** `pm2 delete harmonydesk && pm2 start /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs`. Verify `pm2 env <id>` fingerprint == `.env` fingerprint.
- **Block var (in the `env:` block, e.g. `RESEND_API_KEY`):** `pm2 reload /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs` (PEL-015/017).
- **Classify first:** `grep` the var in the ecosystem `env:` block. In-block → reload. Not-in-block → recreate.
- **Verify at the env layer** (`pm2 env` fingerprint vs `.env`) and only then behaviorally — never behaviorally alone during a dual-secret transition window.

## What NOT to waste time on

- `pm2 restart`, `pm2 restart --update-env`, or `pm2 reload <file>` to refresh a non-block var. None work.
- Treating a passing webhook/auth check as proof of a new secret while the old secret is still inside its expiry window.
- Trusting `pm2 env` "absence" claims from PEL-016 — the var IS there.

## When to apply again

- Every future `.env` secret rotation on HD Pro or any DuelTech project using the `ecosystem.config.cjs`-runs-dotenv-at-parse-time pattern.
- Note: a recreate resets the PM2 restart counter to 0 and assigns a fresh PID — expected, not a fault.
