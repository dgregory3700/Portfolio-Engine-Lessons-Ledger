# PEL-016 — Non-Block Env Vars Refresh on Any Process Respawn; `reload <file>` Is Only for Ecosystem env-Block Vars

**Date logged:** June 6, 2026
**Project:** HarmonyDesk Pro (EC2 backend, PM2 process management — Cat 3G `STRIPE_WEBHOOK_SECRET` rotation)
**Related entries:** Confirms empirically the prediction made in PEL-015's "Diagnostic technique" §1 (non-block vars are loaded by `server.js`'s own `dotenv.config()` at every spawn, so a plain restart already refreshes them at the application layer). PEL-015 stated this as theory; Cat 3G is the first real non-block secret rotation to test it. Sits in the Cat 3 security-rotation arc (Cat 3E `STRIPE_SECRET_KEY`, Cat 3G `STRIPE_WEBHOOK_SECRET`). Next in queue: `RESEND_API_KEY` (the one block-referenced secret — uses PEL-015's `reload <file>`) and `SUPABASE_SERVICE_ROLE_KEY` (non-block — uses this PEL).

---

## Symptom / context

Cat 3G rotated `STRIPE_WEBHOOK_SECRET`. The open question carried in the SSP from Cat 3F was whether this var needed the PEL-015 `pm2 reload <ecosystem-file>` doctrine or whether a plain restart would suffice. Two facts were established by direct inspection of the EC2 box (`/apps/harmonydesk/harmonydesk-mvp/`, process `harmonydesk`) before any rotation step:

1. **`STRIPE_WEBHOOK_SECRET` is NOT in the ecosystem env block.** `ecosystem.config.cjs`'s `apps[0].env` block carries only `NODE_ENV`, `PORT`, `NODE_OPTIONS`, `EMAIL_SERVICE`, `RESEND_API_KEY`, `EMAIL_FROM`, `APP_URL`, `SENDGRID_API_KEY`, `SENDGRID_FROM_EMAIL`, `DEMO_INBOX`. `STRIPE_WEBHOOK_SECRET` lives only in `.env` (line 17).

2. **The secret reaches the process via `server.js`'s own runtime dotenv, not via PM2 injection.** `server.js` line 18 `import dotenv from "dotenv";`, line 30 `dotenv.config();`. `routes/stripe.js:423` reads `const secret = process.env.STRIPE_WEBHOOK_SECRET;` *inside the request handler* (per-request), not captured into a module-level const. The handler today verifies live Stripe signatures, which is itself proof the secret reaches `process.env` despite being absent from the PM2 env block — confirming the runtime-dotenv path.

This closes the question logically before testing: because the var is loaded by the app's own `dotenv.config()` at spawn (not by PM2's cached env block), **any** process respawn re-reads `.env` from disk and repopulates `process.env` with the new value. The two-layer PM2 cache that PEL-015 documented is irrelevant to this var class — the var never passes through either PM2 cache layer.

## Root cause / mechanism

PEL-015 established that PM2 caches env at two layers (daemon `process.env` and the evaluated per-app env block), and that only `pm2 reload <ecosystem-file>` invalidates both. That entire mechanism applies **only to vars referenced in the ecosystem env block.**

A non-block var follows a completely separate path:

- It is never read by `ecosystem.config.cjs`'s env block, so PM2 never stores it in the app definition.
- It is never set in the spawn env by PM2, so `pm2 env <id>` will not show it (and its absence from `pm2 env` is expected, not a fault — see PEL-015 §1).
- It is loaded by `server.js`'s `dotenv.config()` call, which runs fresh in the **spawned application process** every time that process boots.
- Therefore any operation that respawns the application process — `pm2 restart`, `pm2 restart --update-env`, `pm2 reload <name>`, `pm2 reload <file>`, or a teardown cycle — causes `server.js` to re-execute `dotenv.config()` and pick up the current `.env` contents.

The PM2 cache layers are simply not in the path for non-block vars. The cheapest respawn is sufficient.

## Diagnostic technique / verification

The rotation was executed and verified as follows:

1. **Before-fingerprint locked.** Captured the last 6 chars of the current secret (`…LcRJbv`) via `grep … | sed` — low-exposure before/after baseline, same methodology as Cat 3E. Never echoed the full secret to chat.

2. **Source-side roll (operator's hands, Stripe dashboard).** Rolled the signing secret with a 24-hour delayed expiry on the old secret (Stripe's transition window — multiple secrets active, one signature generated per secret until expiration).

3. **`.env` edit.** Backed up `.env` (`.env.bak.cat3g-<epoch>`), then `sed -i 's|^STRIPE_WEBHOOK_SECRET=.*|STRIPE_WEBHOOK_SECRET=<new>|' .env`. Verified exactly one matching line and that the after-fingerprint flipped to the new tail (`…5XSc9t`) **before** any respawn.

4. **Respawn.** Process respawned (fresh PID confirmed: 1114426 → 1131683, restart count 4 → 5). On boot, `server.js`'s `dotenv.config()` re-read `.env` → `process.env.STRIPE_WEBHOOK_SECRET` = new value.

5. **Live-traffic verification — the conclusive step.** A real Stripe event was re-delivered (operator used the dashboard **Resend** on a past `invoice.payment_failed` event) at 17:59:30 UTC, ~5 min post-respawn. Server logs showed `signature_verified` for that event after the boot marker, followed by correct `duplicate_event_ignored` dedup; the Stripe dashboard showed **HTTP 200 `{"received": true}`**. Because the running process held **only** the new secret in `process.env`, a 200 can occur only if the delivery carried a signature computed with the new secret — a 400 ("no matching signature") would have proven failure. The 200 is therefore conclusive proof the new secret is active and the handler verifies against it.

**Caveat on the exact command used:** the respawn this session was run as `pm2 restart harmonydesk --update-env`, not literally-plain `pm2 restart`. For a non-block var this distinction is immaterial: the refresh comes from the application's own `dotenv.config()` on respawn, not from PM2's env injection, so the `--update-env` flag had no bearing on the outcome. The confirmed claim is "any respawn refreshes a non-block var," which plain `pm2 restart` satisfies.

## Canonical fix / pattern

**For rotating a non-block env var (one NOT listed in `ecosystem.config.cjs`'s `apps[0].env` block) on HarmonyDesk Pro or any DuelTech project using the same `server.js`-loads-its-own-dotenv pattern:**

```
# 1. backup + swap the single line in .env (verify fingerprint flips, single match)
# 2. plain respawn — sufficient:
pm2 restart harmonydesk
```

No `pm2 reload <ecosystem-file>` is required for this var class; that doctrine (PEL-015) is reserved for block-referenced vars (`RESEND_API_KEY` is the only such secret in the rotation queue). `pm2 env <id>` will NOT reflect a non-block var — verify at the application layer (live traffic / behavior) instead, not via `pm2 env`.

**Verification for webhook-secret rotations specifically:** use the Stripe dashboard **Resend** on any recent past event rather than "Send test webhook." Stripe re-signs each delivery attempt (including resends) with the endpoint's currently-active secret(s) at send time, so a Resend against a process holding only the new secret is a clean, real-traffic proof. A 200/`signature_verified` is conclusive; the absence of a visible "old" secret in the dashboard Overview is NOT a fault — after a delayed roll, the Overview shows only the current secret while the old one expires automatically at the chosen window.

## What NOT to waste time on

- **Do not reach for `pm2 reload <file>` or a teardown cycle for non-block vars.** They work (any respawn works) but add ceremony for no benefit. Plain restart is the operation.
- **Do not use `pm2 env <id>` to verify a non-block var.** It reports PM2's spawn env, which never contains the var. A "missing" result there is expected and proves nothing. Verify behaviorally.
- **Do not panic when the Stripe dashboard shows only one signing secret after a delayed roll.** That is normal. The roll's success is proven by live verification (the 200), not by the UI surfacing two secrets.
- **Do not re-roll to "force" early expiry of the old secret.** A delayed roll has no after-the-fact early-expire control; re-rolling with "immediate" would kill the just-deployed new secret. Let the chosen window auto-expire.

## When to apply again

- Any future non-block secret rotation: notably `SUPABASE_SERVICE_ROLE_KEY` (next-but-one in the Cat 3 queue) is non-block → plain `pm2 restart` + behavioral verification.
- Any rotation where the secret is consumed via `server.js`'s runtime `dotenv` rather than the ecosystem env block — confirm block-membership first (`grep` the var in `ecosystem.config.cjs`'s env block), then choose: in-block → PEL-015 `reload <file>`; not-in-block → this PEL (plain restart).
- Any webhook-secret rotation across any provider that re-signs deliveries at send time: prefer a Resend of real traffic over a synthetic test event for verification.

## Related context

This PEL is the empirical closure of a prediction PEL-015 made but did not test. It validates the broader principle from PEL-015 — describe mechanism, not just symptom — by showing that once the mechanism (which cache layer, if any, the var passes through) is understood, the correct operation follows deductively and the heavier remediation can be skipped with confidence. The Cat 3F empirical-verification-cycle discipline was not needed here precisely because the mechanism analysis was conclusive on its own; the live Resend served as confirmation rather than discovery.

Secondary methodology note: the Stripe **Resend**-as-verification technique generalizes. Any time a verification depends on "is this newly-deployed secret the one the remote system is signing with," a real re-delivery (re-signed at send time) against a process that holds only the new secret yields a binary, UI-independent answer: 200 = active, 400 = not. This is more reliable than reading dashboard state, which may not surface transitional/expiring credentials.
