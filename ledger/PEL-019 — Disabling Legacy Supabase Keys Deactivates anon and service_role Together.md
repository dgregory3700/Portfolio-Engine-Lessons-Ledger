# PEL-019 — Disabling Legacy Supabase Keys Deactivates anon AND service_role Together; Migrate the anon/Publishable Key Before Disabling

**Date logged:** June 9, 2026
**Project:** HarmonyDesk Pro — Cat 3 `SUPABASE_SERVICE_ROLE_KEY` migration (close-out of the Cat 3 breach-rotation arc)
**Related entries:** Cat 3 security-rotation arc. Pairs with PEL-018 (the PM2 refresh correction surfaced in the same session). PEL-012 discipline applied (verify, don't trust assertions).

---

## Symptom / context

The Cat 3 goal was to retire the breach-flagged legacy `service_role` JWT. After migrating it to `sb_secret_` on both surfaces (EC2 `services/supabase.js`; Vercel `publicSupabase.ts`, `lib/supabase/admin.ts`, `supabaseServer.ts`), the final step was to disable the legacy key at source. The trap: the dashboard's own auth depends on the legacy `anon` key, which the service_role rotation never touched.

## Root cause / mechanism

- Legacy `anon` and `service_role` are both JWTs validated off the project's shared legacy JWT secret. Supabase's "disable legacy API keys" deactivates **both at once** — confirmed by the post-disable rejection `"Legacy API keys are disabled"`.
- The HD Pro dashboard auth clients (`src/lib/supabase/client.ts` `createBrowserClient`, `src/lib/supabase/server.ts` `createServerClient`) both read `NEXT_PUBLIC_SUPABASE_ANON_KEY`. On the live Vercel deployment this was still a legacy `anon` JWT (confirmed by grepping the deployed client bundle for an `eyJ…` token).
- Therefore disabling legacy to kill the leaked `service_role` would simultaneously 401 the only operator's login. Self-lockout.

## Diagnostic technique / verification

- **Enumerated every consumer of the key before swapping** (PEL-012): the in-code comment in `publicSupabase.ts` claimed it was the *only* service-role holder — wrong; `search_code` + a direct directory listing found three.
- **Determined the live anon key type without Vercel access** by scanning the deployed Next.js client bundle (`/_next/static/...js`) for `sb_publishable_` vs `eyJ…`. `NEXT_PUBLIC_` vars are inlined into the public bundle, so the key type is observable from the served JS. Pre-migration: legacy JWT in shared chunk; post-migration: `sb_publishable_`, no JWT.
- **Isolate-tested both new keys** (createClient + a real read) before committing them to `.env`/Vercel — never swap a key you haven't proven authenticates.
- **Post-disable smoke test (the real proof):** new `sb_secret_` accepted; **old legacy JWT rejected** with "Legacy API keys are disabled"; public page still renders; incognito dashboard login still works.

## Canonical fix / pattern

To retire a leaked legacy `service_role` key on a project that also serves a legacy `anon` key:
1. Create new `sb_secret_` (server) and `sb_publishable_` (browser) keys; leave legacy active.
2. Swap every server consumer to `sb_secret_`; swap `NEXT_PUBLIC_SUPABASE_ANON_KEY` to `sb_publishable_` (Sensitive OFF — public by design). Redeploy.
3. Verify: bundle no longer contains an `eyJ…` JWT; incognito login works; server reads succeed on `sb_secret_`.
4. **Only then** disable legacy. Confirm the old leaked JWT is rejected.

## What NOT to waste time on

- Assuming legacy `service_role` can be disabled independently of legacy `anon`. It can't — they go together.
- Trusting a code comment about which file holds a key (verify by grep + directory listing).
- Marking `NEXT_PUBLIC_SUPABASE_ANON_KEY` "Sensitive" on Vercel — it's a public key; Sensitive is for the secret key only.
- Worrying about Bearer incompatibility of the new keys in app code: `@supabase/supabase-js` and `@supabase/ssr` (installed versions) handle `sb_secret_`/`sb_publishable_` natively. Only a raw REST `Authorization: Bearer` would break — none exists in either HD Pro repo.

## When to apply again

- Any other DuelTech project still on legacy Supabase keys (legacy sunset is late 2026).
- HD Pro is now fully on `sb_secret_` (service-role) + `sb_publishable_` (anon); legacy disabled 6/9/26.
