# PEL-024 — Enabling RLS on Live Tables: the Bare "Enable RLS" Auto-Remediation Default-Denies the Dashboard; Mirror an Already-RLS-On Sibling's Policy, and Confirm Each Public Route's Client Is Service-Role Before Flipping

**Date logged:** June 25, 2026
**Project:** HarmonyDesk — post-SSP #3 security carryover (RLS on the six `rls_disabled_in_public` tables; YELLOW under Execution Doctrine v1.2)
**Related entries:** PEL-023 (trace the WRITE path / don't trust the plan; price-gate semantics), PEL-019 (legacy Supabase key disable hits anon + service_role together), PEL-020 (derive verdicts on the consuming surface), PEL-001 (Supabase RPC / SECURITY DEFINER behavior).

---

## Symptom / context
The six `public` tables flagged `rls_disabled_in_public` (ERROR) — `clients`, `messages`, `invoices`, `subscriptions`, `user_settings`, `counties` — were the last real-risk carryover on HarmonyDesk. RLS off + the default anon/authenticated grants in a PostgREST-exposed schema = anyone holding the publishable key (which ships in the frontend bundle) can read/write every row. Five of the six hold PII / billing / confidential mediation data.

The naive fix — Supabase's one-click `ALTER TABLE … ENABLE ROW LEVEL SECURITY` — is a live-product trap: RLS with **no policy** is default-deny, so every cookie-bound authenticated dashboard read returns zero rows and the customer's dashboard goes blank. The advisor payload says so itself ("enabling RLS without policies will block all access").

Subtler trap: `user_settings` is read by the **public** `/request/<slug>` route. Had that route read it as anon, an authenticated-only policy would have silently broken the public intake funnel that feeds the entire SP-SOP signup pipeline.

## Root cause / mechanism
- RLS is the *only* gate once anon/authenticated hold table grants (the Supabase default). Toggling RLS on without a policy doesn't usefully secure the table — it locks out the legitimate authenticated client too. Service-role bypasses RLS, so only service-role paths keep working.
- The correct fix already existed in the same schema. `cases` is `user_email`-keyed and **already RLS-on**, read by the dashboard daily. Its 4-policy set (role `authenticated`, `user_email = (auth.jwt() ->> 'email')` for SELECT/INSERT/UPDATE/DELETE) is the proven template for the six `user_email`-keyed targets. Newer tables (`sessions`, `intakes`, `document_templates`, `generated_documents`) use the parallel `user_id = auth.uid()` template.
- Whether enabling RLS breaks a public surface depends entirely on **which client that surface uses.** Reading `src/lib/publicSupabase.ts` proved `getMediatorBySlug` builds its client with `SUPABASE_SERVICE_ROLE_KEY` (service-role → RLS-exempt); the file comment even documents the deliberate choice to use service-role *specifically to avoid* coupling the public page to `user_settings` RLS. Cat 2F's public POST runs on EC2 with its own service-role resolution. No public path touches these tables as anon → authenticated-only policies are safe.

## Diagnostic technique / verification
Reusable RLS-enable derivation for any live table:
1. **Don't run the bare ENABLE.** For each target, find an **already-RLS-on sibling with the same ownership key** and copy its policy verbatim (`SELECT … FROM pg_policies WHERE tablename IN (…)`). Here: `cases` for the `user_email` pattern.
2. **Per-table deviation, not blind copy.** Scope writes to who should actually write. `subscriptions` → SELECT-own *only* (the EC2 webhook writes it via service-role; authenticated UPDATE would let a user flip `status` to `active` and self-upgrade past `UpgradeWall`). `user_settings` → SELECT/INSERT/UPDATE-own, **no DELETE** (one row/user, UNIQUE `user_email`).
3. **Before flipping RLS on any table a public route reads, read that route's data-fetch and confirm the client is service-role (or a SECURITY DEFINER RPC), not anon.** Read the actual `createClient(…)` key — a stale comment is not proof.
4. **Apply atomically** via `apply_migration` (one migration; DDL in a transaction → all-or-nothing, auditable in `list_migrations`).
5. **Verify on absence-of-INFO, not just absence-of-ERROR.** Post-apply `get_advisors(security)`: the six `rls_disabled_in_public` ERRORs clearing proves RLS is on; the six **not** appearing under `rls_enabled_no_policy` (the way correctly-locked `password_setup_tokens` / `stripe_webhook_events` still do) proves each got a policy. ENABLE-without-policy would surface there — so its absence is the positive signal.
6. App-side: authed operator loads Clients/Billing/Messages/Settings and confirms render. Rollback is instant, per-table (`DISABLE ROW LEVEL SECURITY` + `DROP POLICY`).

## Outcome (June 25, 2026)
- Migration `enable_rls_public_tables_owner_policies` applied to `ziprlzqjssilwhpsjizn` (`harmonydesk-billing`). RLS + owner policies on all six: `clients`/`messages`/`invoices`/`counties` (full CRUD-own), `subscriptions` (SELECT-own), `user_settings` (SELECT/INSERT/UPDATE-own).
- Advisor re-run: **zero ERROR lints**; the six absent from `rls_enabled_no_policy` → policies confirmed attached. Exposure closed.
- Left in place by design (non-ERROR): `get_user_id_by_email` anon/authenticated EXECUTE (WARN — load-bearing in the public POST flow); four `function_search_path_mutable` WARNs; `password_setup_tokens`/`stripe_webhook_events` RLS-on-no-policy INFO (correct service-role-only posture).

## What NOT to waste time on
- Don't run Supabase's one-click "Enable RLS" on a table the app reads as `authenticated` — it default-denies and blanks the surface. The remediation SQL is half a fix.
- Don't invent policies. If a same-ownership-column sibling is already RLS-on and working, copy it — it's proven against the live client.
- Don't blind-copy full CRUD onto every table — service-role-written tables (subscriptions) want SELECT-own at most; authenticated writes can open privilege escalation.
- Don't assume a public route's client; read the `createClient` key, not the comment.
- Don't add an anon SELECT policy "to be safe" on `user_settings` — its public path is already service-role/RLS-exempt; an anon policy would re-widen the exposure you just closed.

## When to apply again
- Any `rls_disabled_in_public` ERROR on any DuelTech Supabase product (the 20+ portfolio will hit this on older tables predating the `user_id`/`auth.uid()` convention).
- Any time you enable/modify RLS on a table a public/unauthenticated route also reads — confirm that route's client first.
- Any Supabase advisor burn-down: use the absence of `rls_enabled_no_policy` as positive confirmation that policy creation (not just the RLS toggle) succeeded.
