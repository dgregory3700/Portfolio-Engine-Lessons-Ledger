# PEL-023 — A Price-Gate Cutover Is Safe Only If the WRITE Path Emits the Gated Literal; Trace the Surface That Writes `price_id`, Don't Trust the Plan

**Date logged:** June 23, 2026
**Project:** HarmonyDesk — SSP #2 (single-product Stripe cutover; first all-YELLOW run under Execution Doctrine v1.2)
**Related entries:** PEL-020 (derive verdicts on the consuming surface, not the one a var points at), PEL-012 (verify prior assertions, don't trust them), PEL-014 ("Connected"/"running" status badges are not authoritative), PEL-001 (Stripe webhook price handling).

---

## Symptom / context

SSP #2 consolidated to one $99 product by KEEPING `prod_UFySWw8kh1BUfR` on its existing keystone price `price_1THSVD819PQIqsMidaht2zmZ`, renaming it, and archiving the $49 "Mediator Edition" (`prod_U4uaecxqFQsAsr`). The dashboard `UpgradeWall` admits a user **iff** `subscriptions.price_id == HD_PRO_PRICE_ID` (the literal `price_1THSVD…`, hardcoded in `(dashboard)/layout.tsx`, queried `.eq('price_id', HD_PRO_PRICE_ID).in('status',['active','trialing'])`). The SSP's KEYSTONE GUARD said "keep the price; provisioning should be unaffected — verify, don't assume."

Two stale-premise risks surfaced at session open:
- **(a)** EC2 `.env` carried `STRIPE_PRICE_ID=price_1Su1t4819PQIqsMiuPzloa43` — a **third** price id matching neither product's current default price (keystone `price_1THSVD…` nor Mediator `price_1T6klj…`). On its face this looked like a gate-lockout landmine.
- **(b)** The SSP assumed **zero** active/trialing subs (basis for "no customer migration"). Live data showed **1 active sub** (`duelgregory@yahoo.com`, the owner account) on the keystone price.

## Root cause / mechanism

- A price-gated cutover's safety hinges entirely on **what value the WRITE path emits into `subscriptions.price_id`** — not on env vars that merely look price-shaped. Reading `routes/stripe.js` proved `priceId` is sourced from the **live Stripe subscription object** (`sub.items.data[0].price.id`, with `expand: ['items.data.price']`) in both the checkout-completed and subscription-updated handlers. So as long as the surviving link rides the keystone price, every new sub writes the gate-matching literal. Gate-safe by construction.
- `STRIPE_PRICE_ID` (the scary third price) is referenced **nowhere in code** (`grep -rI --exclude=".env*" STRIPE_PRICE_ID` → empty). It is a dead env var — same category error as PEL-020. Its *value* is irrelevant because nothing reads it.
- The "zero subs" premise was **stale** (set June 20, not re-verified). Live = 1 sub, but on the **surviving** keystone price → harmless to the cutover, and it doubles as the ideal post-cutover `UpgradeWall` smoke subject. The premise being wrong didn't change the conclusion — but only because live derivation caught it. Trusting it would have left a real blind spot.

## Diagnostic technique / verification

Reusable gate-safety derivation for **any** price-gated cutover:
1. Find the gate literal and its exact query (dashboard layout: `.eq('price_id', HD_PRO_PRICE_ID)`).
2. **Trace the WRITE path** that populates that column (here the EC2 webhook). Confirm it emits the value from the **live provider object**, not a local const/env. Read the handler; don't trust the SSP's "should be unaffected."
3. For any price-shaped env var, grep the code for its **NAME** (`--exclude=.env*`) before reasoning about its value. Unreferenced ⇒ red herring (PEL-020).
4. Re-derive subscriber state live (Stripe + `subscriptions` table) at session open. Never carry "zero subs" from a prior session.
5. Post-cutover smoke: re-query `subscriptions` for active/trialing on the gate price; confirm the gate still admits.

**Stripe MCP op-id reference for this server** (saves trial-and-error next time):
- create payment link → `PostPaymentLinks`; update/deactivate link → `PostPaymentLinksPaymentLink` (set `active`); list links → `GetPaymentLinks`; link line items → `GetPaymentLinksPaymentLinkLineItems`.
- update/archive product → `PostProductsId` (set `name`/`active`). **`PostProductsProduct` does not exist** — first guess failed.
- reads: `fetch_stripe_resources` (by id) and `search_stripe_resources` (`resource:clause`, e.g. `products:active:"true"`).

**Lapsed-auth signature:** Stripe + Supabase MCPs in a lapsed-auth state return `"No approval received"` on **every** call including reads. Fix = reconnect in Customize → Connectors. A successful read is the only authoritative confirmation; the "Connected" label can lie (PEL-014). *(Qualified June 24 — see Addendum: "No approval received" can also be PER-TOOL while the connector is live.)*

## Outcome (June 23, 2026)

- **Final Stripe state:** ONE active product "HarmonyDesk" (`prod_UFySWw8kh1BUfR`) / `price_1THSVD819PQIqsMidaht2zmZ` ($99) / ONE active link `plink_1TlXSr819PQIqsMiAMSCi6QU` (`buy.stripe.com/8x2fZbcWW9zfdqqfBkdZ602`; 14-day trial; `trial_settings.end_behavior.missing_payment_method=pause`, matched to the old link). **Archived:** `prod_U4uaecxqFQsAsr` (Mediator); its $49 link `plink_1T7Hsr819PQIqsMiQEWozSWx` deactivated. Old link `plink_1SuCSJ…` was already inactive.
- **Repoints (both verified live):** public `harmonydesk.ai` (3 buy hrefs in `index.html`; `affiliates.html` had none) and `harmonydesk.dev` demo (`NEXT_PUBLIC_STRIPE_CHECKOUT_URL`) → the $99 link.
- **Gate intact:** 1 active sub on `price_1THSVD…` post-cutover; `UpgradeWall` still admits.
- **Carryover:** dead EC2 `STRIPE_PRICE_ID=price_1Su1t4…` (hygiene delete — derive per PEL-020 first); Phase B single-$99 landing rewrite (the literal "$49/mo" copy still on `harmonydesk.ai`); SSP #3 demo fork re-sync.

## What NOT to waste time on

- Don't panic over a price-shaped env var until you've grepped the code for its **name** — an unreferenced `STRIPE_PRICE_ID` is inert regardless of value (PEL-020).
- Don't trust a prior session's "zero subs" / live-state claim; re-derive at open (PEL-012).
- Don't try to fetch-verify a Vercel preview/deploy guarded by **Deployment Protection (SSO)** — even the Vercel MCP's `web_fetch_vercel_url` and the `get_access_to_vercel_url` share link return the 401 auth wall. Verify the **committed source** instead (static site: committed == served) and have the authed operator eyeball the protected preview in-browser.
- Don't assume the connected Vercel MCP can set env vars — it exposes reads/deploys/logs only. `NEXT_PUBLIC_*` changes require a dashboard edit **plus** a rebuild (build-time inlined), and a code fallback can't override a set env var.

## When to apply again

- Every price/plan migration, product consolidation, or price-gate change on any DuelTech product whose subscription gate keys on `price_id`.
- Any cutover whose plan asserts "no migration / zero subs / provisioning unaffected" — treat each as a hypothesis to confirm by tracing the write path, not a given.

---

## Addendum — June 24, 2026 (SSP #2 Phase B closeout + verification refinements)

**Phase B shipped — the landing carryover above is now closed.** `harmonydesk.ai/index.html` was rewritten to a single $99 product, grounded in the LIVE pro-dashboard nav (`harmonydesk-pro-dashboard` @ `ad0eea2`, `DashboardShell.tsx` + page source) rather than memory or the archived backup; PR #2 squash-merged to `main` (`f143473`). The old "$49/mo" copy and the cheaper-tier narrative are gone. `affiliates.html` positioning was synced in the same pass (PR #3, squash `983af6c`) — feature copy + the shareable positioning line now match the single-$99 dashboard story; still no price shown (intentionally routes traffic to the landing for Rewardful attribution). Both pages verified live. SSP #2 is fully complete; the queue advances to SSP #3 (demo fork re-sync). Net positioning call worth remembering: the landing now mirrors only the **surfaced** dashboard nav and deliberately drops "Case tracking" as a headline, because Cases/Intake/Booking-links are hidden from the live nav (attorney-coded); and the repo's own blurb ("SMS, Cal.com, AI drafting") is stale — none of those exist, so they were excluded.

**Refinement 1 — `"No approval received"` is NOT always a whole-connector lapsed-auth state; it can be PER-TOOL while the connector is live.** This qualifies the "Lapsed-auth signature" note above. During Phase B, `Vercel:web_fetch_vercel_url` returned `"No approval received"`, yet `Vercel:list_deployments` succeeded **in the same session, on the same connector**, returning real deployment data. So a single tool's refusal does **not** prove the connector is down. Corrected test for connector liveness: if **any** read tool on that connector returns real data, the connector is live — treat a sibling tool's `"No approval received"` as a per-tool approval/permission issue, **not** a reconnect trigger. Only reconnect (Customize → Connectors) when **every** call including reads fails. Practical rule: before reconnecting on a refusal, probe a second tool on the same connector first; don't tear down a live connection because one tool balked. (Note this is also distinct from the SSO 401 wall `web_fetch_vercel_url` returns on protected previews — three different failure modes for the same tool: 401 SSO wall, per-tool "No approval received", and true connector-wide lapsed-auth.)

**Refinement 2 — Post-deploy verification against the public URL can be fooled by edge cache; reconcile against committed source + deploy state, not a re-fetch.** After PR #3 merged, the Vercel prod deploy reached `READY` (confirmed via `list_deployments`: commit `983af6c`, `target: production`), yet a `web_fetch` of the public `harmonydesk.ai/affiliates.html` kept returning the **pre-merge** copy — a CDN/fetch-layer cache serving a response captured seconds before the build finished. Re-fetching the same URL did not bust it. Authoritative confirmation for a static-site deploy = **committed source on `main`** (read via GitHub MCP; committed == served) **+ the Vercel deploy showing `READY` on the production target**, with the authed operator hard-refreshing (Cmd/Ctrl-Shift-R) in-browser to see it. Do **not** read a stale public fetch as a failed deploy and re-ship; the deploy was correct and live, the fetch was cached. (This extends the existing "verify committed source, committed == served" guidance from the SSO bullet above to the *post-merge public-URL* case.)
