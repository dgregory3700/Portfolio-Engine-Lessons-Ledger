# PEL-017 — Block Env Vars Refresh on pm2 reload of the Ecosystem File and Show in pm2 env

**Status:** Confirmation entry (validates PEL-015 on a live block secret)
**Date:** 2026-06-09
**Context:** HarmonyDesk Pro — Cat 3A security rotation (RESEND_API_KEY)
**Related:** PEL-015 (reload doctrine), PEL-016 (non-block contrast)

## Quick recognition test
You are rotating a secret that lives in the `apps[0].env` block of `ecosystem.config.cjs`, and you need to prove the running PM2 process actually took the new value — not just that the file on disk changed.

## Root cause / context
Not a bug. This is the first LIVE application of PEL-015's reload-from-absolute-file path on a real production block secret. Prior validation of the reload path used only the throwaway DEMO_INBOX probe var. Cat 3A (RESEND_API_KEY) was the first — and, for HD Pro, the last — real block-referenced secret to exercise it.

## What was confirmed
1. `pm2 reload <absolute-path-to-ecosystem.config.cjs>` DOES refresh a block var. Mechanism: `ecosystem.config.cjs` runs `dotenv.config({ path: path.join(__dirname, ".env") })` at file-read time, so when PM2 re-reads the config the `.env` is re-sourced, the env block re-evaluates `RESEND_API_KEY: process.env.RESEND_API_KEY`, and the respawn injects the new value.
2. For a block var, `pm2 env <id> | grep <VAR>` REFLECTS the new value post-reload, because the value is injected into the process environment at spawn. This is a clean env-layer proof.
   - CONTRAST (PEL-016): non-block vars are read at runtime by `server.js`'s own `dotenv.config()`, so they are NOT in PM2's injected env and `pm2 env` is blind to them. The Cat 3G verification had to lean on a live Stripe Resend because `pm2 env` could not see `STRIPE_WEBHOOK_SECRET`.
3. Empirical markers (Cat 3A): `pm2 env 0` showed the new `RESEND_API_KEY` tail immediately after reload; process PID 1131683 → 1152863, restart 5 → 6; transactional email verified through both the live EC2 send path (request + approval emails to client and mediator) and the redeployed Vercel functions before the old key was deleted.

## What NOT to waste time on
- Do not use `pm2 restart --update-env` to refresh a `.env` change — it replays the daemon's cached env, it does not re-read the file (PEL-015).
- Do not assume `pm2 env` is blind to the var. That is only true for non-block runtime-dotenv vars (PEL-016). For a block var, `pm2 env` is your fastest first confirmation.
- Do not delete the old key at the provider until every surface is verified on the new key. Resend keys have NO auto-expiry (unlike Stripe's delayed-roll webhook secret) — deletion is manual, immediate, global, irreversible.

## Canonical fix / procedure
1. Back up `.env`; swap the line; verify single line + fingerprint flip BEFORE respawn.
2. `pm2 reload /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs` (absolute path).
3. Confirm at env layer: `pm2 env 0 | grep <VAR>` shows the new value.
4. Confirm functionally: one live send through each consuming surface.
5. Only then delete the old key at the provider.

## When to apply again
Any ecosystem env-block secret rotation. As of Cat 3A, `RESEND_API_KEY` was the last block-referenced secret in HD Pro; all remaining rotations (`SUPABASE_SERVICE_ROLE_KEY`, etc.) are non-block → plain `pm2 restart` per PEL-016, with `pm2 env` blind to them.
