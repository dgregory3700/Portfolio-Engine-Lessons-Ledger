# SaaS Operating System
**Execution Doctrine v1.2 — Autonomy & Reversibility Guardrails**

Version 1.2. **Extends** Execution Doctrine v1.1; **supersedes only** v1.1's
"Production deployments are the test harness" stance (§I), and that same
spirit where it recurs in v1.1 §III (Validation Loop) and §V (Scope
Philosophy). Every other v1.1 section — §I tool layer, §II code handling,
§IV pipeline, §VI SSP, §VI-B handoff, §VII authority, §VIII change control —
remains in force unchanged.

Authored June 20, 2026. Applies across all DuelTech projects and all AI
execution tools (Claude, ChatGPT, Copilot, Gemini, Grok).

> The numbering below (§1–§9) is internal to v1.2 and does not renumber v1.1.

---

## 1. Why v1.2 exists

v1.1 treated production as the test harness — workable when a human ran every
step and reviewed each change as it happened. Under the current model, agents
read, decide, and write to production, and the director's time is the scarce
resource. "Review after" is therefore unreliable as a *safety* mechanism: by
the time it happens, the action is already live.

v1.2's thesis: **safety lives in the envelope, not the review.** The agent acts
autonomously precisely because the path it acts through is reversible and
self-verified — not because a human watches each step.

**CORE REVISION (supersedes v1.1 §I bullet):**
"Production deployments are the test harness"
  →  **"Preview/staging is the test harness; production is the verified
     destination."**
Production is where verified changes land, not where they are tested.

---

## 2. Operating model

**"Duel as director, agents as executors."**
- The bottleneck to remove is the manual integration toil (copy-paste-verify
  between AI and live systems) — not the director's judgment.
- Stack rule: drop any account/tool that cannot connect to the AI agents.

**AUTONOMY DIAL = AUTONOMOUS.** The agent reads, decides, and writes — to
production — within the reversibility envelope below. Autonomy is not
unconditional; it scales with reversibility (§3).

---

## 3. The reversibility tiers (autonomy scales with reversibility)

Every side-effecting action is classified before it is taken. When in doubt,
classify **up** (toward red), never down.

**▸ GREEN — fully reversible, git/config-revertable, no blast radius.**
- GATE: none. Agent proceeds, reports after.
- Includes: branch creation; code edits on a branch; reads/inspections via any
  MCP; PEL and doc writes to the ledger; preview deploys.

**▸ YELLOW — reversible-with-a-net. Permitted, but only *through* the net.**
- GATE: the net + a single promote-confirm (§4).
- Includes: production code deploys; reversible config (Stripe product/link
  create · rename · archive · deactivate); non-secret functional env-var
  add/edit; Supabase schema changes shipped WITH a written down-migration +
  fresh backup; outbound email (rate-capped).

**▸ RED — irreversible or outside the envelope.**
- GATE: HARD STOP. The agent never performs these. It spells out the exact
  action and hands it to the director.
- Includes: hard deletes & emptying trash; table drops; money movement
  (charges, refunds, transfers, payouts); credential/secret rotation;
  infrastructure destruction (terminate EC2, delete projects); DNS changes;
  access-control / permission / RLS-widening changes; OAuth/SSO grants.
- Note: secret-valued env vars are RED (they belong to rotation); only
  non-secret functional config env vars are YELLOW.

---

## 4. The yellow procedure (the only human checkpoint in the envelope)

Two settled rules govern every yellow change:

**DECISION 1 — Promote-confirm, NOT auto-promote.**
The agent carries a change all the way to a green, self-verified preview, then
pauses for the director's one-word "go" before merging to production. No
auto-promote. The confirm is the single human checkpoint; everything around it
is automated.

**DECISION 2 — Promote-then-smoke for auth-gated smokes.**
The agent's green build + preview self-checks are sufficient to promote. When
the functional smoke needs the director's authenticated session, that smoke
runs on PRODUCTION right after promotion, rollback staged. The agent does not
block promotion waiting on a human-only preview smoke.

**Canonical yellow flow (code → production):**
1. Branch → push.
2. Vercel preview → agent self-verifies (deploy READY, build/type clean, any
   agent-runnable smoke on preview).
3. PAUSE → present the change + preview link; wait for the one-word go.
4. On go: merge → production deploy → agent verifies READY on the prod alias.
5. If the smoke needs the director's session, it runs on prod now; rollback
   (§5) staged.
6. Report; log to memory; PEL if a non-obvious lesson surfaced.

**Yellow without a preview surface (Stripe / config / env vars):**
There is no preview to verify against, so the net is reversibility-by-default
+ read-back. The agent (a) states the exact mutation and its one-step
rollback, (b) takes the one-word go, (c) executes, (d) reads the resource back
to confirm. Batch related mutations behind a single confirm only when they
share one rollback.

**DB-migration yellow:** never touch production data without (a) a written
down-migration and (b) a fresh backup first; same promote-confirm. A migration
with no down-migration is RED.

**Email yellow:** sends are rate-capped. Uncapped/bulk sends are RED until
confirmed with a cap in place.

---

## 5. Enabling mechanisms (what makes the envelope hold)

1. **Least-privilege keys** — user-scoped reads/writes use the cookie-bound,
   RLS-aware client. Service-role only where unavoidable, NEVER in a
   user-reachable surface. A service-role import in a user-facing route is a
   security regression.
2. **Reversibility-by-default** — archive over delete, deactivate over destroy,
   down-migration over one-way migration, soft over hard. If the only available
   form of an action is irreversible, it is RED.
3. **Agent self-verification** — every yellow action is verified against live
   state BOTH before the promote-confirm (preview READY / read-back parity) AND
   after (prod READY / read-back).
4. **Action log + fast rollback** — every prod-touching action is recorded
   (commit SHA, deploy ID, config before→after) in the session SSP/handoff.
   Standing rollbacks: Vercel instant rollback to the prior READY prod deploy
   (code); the written down-migration (DB); the inverse config mutation
   (Stripe/env — e.g., unarchive, reactivate, restore prior value).

---

## 6. Classify-before-act (standing obligation)

At the start of any side-effecting action the agent states its tier
(green/yellow/red) and, for yellow, names the net it will pass through and the
rollback it will stage. Red actions are surfaced as a written instruction for
the director — never attempted, never "worked around."

---

## 7. Worked example — Cat UI-FR (the canonical yellow run)

Session-request freshness fix, June 20 2026. Reference implementation of §4:
branch `cat-ui-freshness-fix` → push 3 files → preview `dpl_CZFG…` READY +
agent self-verify (clean build, new route compiled) → PAUSE for go → merge
PR #3 (`ad0eea2`) → prod `dpl_BEJQEc…` READY on pro.harmonydesk.ai →
director's authenticated functional smoke on prod (badge live, Cat 2H emails
fired) → memory logged + PEL-022 written. No red actions; rollback (prior prod
deploy) available throughout.

---

## 8. What carries forward from v1.1 (unchanged)

- Full-file replacement only (no partial diffs).
- Vercel-first; GitHub Web UI / Desktop / MCP; no local dev as authoritative.
- §VI SSP + §VI-B handoff templates required at session open/close; never
  freeform.
- Atomic PEL entries flow direct to this ledger (`ledger/`) via GitHub MCP; the
  LIVE ledger is the source of truth for the next PEL number.
- v1.1 §VII still holds: Execution Doctrine governs *how*; SSP governs *what*;
  Doctrine wins unless the SSP explicitly overrides it.

---

## 9. Applied pre-classification — SSP #2 (single-product cutover)

Under v1.2 the entire cutover is **YELLOW — no red actions:**
- Create a $99 "HarmonyDesk" payment link **on the existing price**
  (price_1THSVD819PQIqsMidaht2zmZ) — reversible config.
- Rename the $99 product → "HarmonyDesk" — reversible (rename back).
- Archive the Mediator product + deactivate its $49 link — archive/deactivate
  are reversible; NOT a delete.
- Repoint buy buttons (landing code) + repoint the demo's checkout env var —
  via the canonical / config yellow flows.

**Gate-integrity guard:** the UpgradeWall paywall in
`(dashboard)/layout.tsx` matches on `HD_PRO_PRICE_ID =
"price_1THSVD819PQIqsMidaht2zmZ"`. Keep the surviving product on that **same
price object** (rename the product, do NOT mint a new price) so the literal
stays valid; otherwise every user is locked out. No money movement is
involved, so nothing in #2 is red.

---

*Execution Doctrine v1.2 — autonomy dial + green/yellow/red reversibility
guardrails. Extends v1.1; supersedes only its production-as-test-harness
stance.*
