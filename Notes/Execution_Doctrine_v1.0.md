# Execution Doctrine v1.0  
**Gregory SaaS Build System**

## Purpose
This doctrine defines how work is executed across all ventures.  
It is independent of any specific product (e.g., HarmonyDesk, ContentVeritas) and governs tooling, workflow, and collaboration discipline.

It exists to maximize:
- Velocity
- Production realism
- Signal over speculation
- Deterministic deployment outcomes

---

## 1. Operating Environment

**Primary Platform**
- Vercel-first
- Production-like deployments are the test harness

**Local Development**
- Avoided by default  
- Used only when a task is impossible in Vercel/GitHub workflows

**Source Control**
- GitHub is the canonical source of truth  
- GitHub Web UI or GitHub Desktop preferred

---

## 2. Code Handling Rules

**File-Level Authority**
- Prefer full-file replacement over partial diffs
- Do not edit or invent file contents without seeing the file

**No Guessing**
- The assistant must ask to see the file before proposing changes
- No speculative directory structures
- No assumed schemas, routes, or imports

**Change Discipline**
- Every change must be:
  - Pasteable
  - Committable
  - Deployable
  - Observable via Vercel logs

---

## 3. Validation Loop

**Canonical Feedback Cycle**
1. Write or update file
2. Commit to GitHub
3. Deploy via Vercel
4. Inspect deployment logs
5. Iterate based on real runtime behavior

This loop is authoritative.  
Local simulations and hypothetical outcomes are not.

---

## 4. SaaS Factory Pipeline

All ventures follow this sequence:

1. Idea validation
2. Domain purchase
3. GitHub repository creation
4. Vercel deployment
5. Supabase integration
6. SendGrid integration
7. Stripe or LemonSqueezy integration
8. Public launch
9. Post-launch hardening

No step is skipped without explicit reason.

---

## 5. External Services

All integrations must be:
- Official SDKs or client libraries
- Production-grade
- Observable via logs and dashboards

Primary stack:
- Supabase
- SendGrid
- Stripe or LemonSqueezy
- Vercel

---

## 6. Collaboration Contract (Assistant Behavior)

The assistant must:
- Ask for files instead of guessing
- Provide full-file outputs when making changes
- Optimize for Vercel deployment success
- Avoid workflows that require local build chains
- Treat deployment logs as first-class debugging input

---

## 7. Scope Philosophy

- Shipping beats polishing
- Production truth beats theory
- Deletion beats complexity
- Observability beats elegance

The objective is to get real users onto real infrastructure with real payments as fast and safely as possible.

---

## Change Control

To modify this doctrine:
- State the change explicitly
- Declare it as a doctrine-level update
- Confirm memory should be updated

Until changed, this doctrine governs all work.
