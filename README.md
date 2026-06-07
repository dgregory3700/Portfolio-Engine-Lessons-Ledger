# Portfolio Engine Lessons Ledger

### Canonical Reference System for Reusable Troubleshooting Knowledge

---

## Purpose (Non-Negotiable)

The **Portfolio Engine Lessons Ledger** exists to:

- Preserve hard-won troubleshooting knowledge
- Prevent repeated investigation of already-solved problems
- Allow rapid rehydration of context in new chat sessions
- Act as a shared memory substrate across sessions and AI collaborators

This is **not** a narrative log. It is **not** conversational. Each entry is **atomic**, **self-contained**, and **reusable**.

> **Rule:** Ledger entries are copied in whole, never paraphrased.

---

## Repository Structure

| Path | Contents |
|---|---|
| `ledger/` | **PEL-000 → PEL-016** — atomic, reusable troubleshooting lessons. The core of this repo. |
| `playbooks/` | Operational SOPs & templates: SP-SOP (+ quick reference), Intake prompt, SSP template, Advisor Access Enablement Checklist. |
| `status/` | Product freeze/resume markers (e.g., EntropyCleaner). *Not* lessons — current-state notes. |
| `Notes/` | Working notes & briefs (ContentVeritas Phase-2 plan, Portfolio Engine foundation brief). |
| `GOVERNANCE.md` | Canonical governance policy (**v2.0**). Binding for all contributors, human or AI. |
| `Execution Doctrine v1.1.md` | Execution doctrine + Session Start Protocol + Session Close/Handoff Protocol (**v1.1**). |
| `Product Discovery Doctrine.md` | Strategic product-discovery doctrine. *(Placement under review — doctrine, not playbook.)* |

> Note: superseded/obsolete files (Execution Doctrine v1.0, the prior v1.0 governance doc, the retired Cal.com integration guide) were removed from the working tree; all remain recoverable in git history.

---

## Entry Structure (Required)

Each ledger entry follows this exact structure:

1. **Ledger ID + Title** (known-issue name)
2. **Quick Recognition Test** (short, human-readable)
3. **Root Cause** (canonical diagnosis)
4. **What NOT to Waste Time On**
5. **Canonical Fix Pattern** (AI-oriented)
6. **When to Apply This Again**

Optimized so a human can recognize the issue in seconds and an AI can consume the body and act without rediscovery.

New entries are added **only** when a problem has been fully understood, correctly diagnosed, and definitively resolved. The intake prompt that generates new entries lives at `playbooks/Intake_Prompt.md`.

---

## Ledger Index

- [PEL-000 — Credit System Idempotency Failure from Unstable Credits Contract](<ledger/PEL-000 — Credit System Idempotency Failure from Unstable Credits Contract.md>)
  _Supabase · credits contract · idempotency · RPC normalization_

- [PEL-001 — Stripe Webhook Returns 500 Due to Ambiguous Supabase RPC Function](<ledger/PEL-001 — Stripe Webhook Returns 500 Due to Ambiguous Supabase RPC Function.md>)
  _Stripe · Supabase · RPC · SQL function design_ — short early stub, worth expanding

- [PEL-002 — EC2 Outgoing Email Booking-Created Flow](<ledger/PEL-002 — EC2 Outgoing Email Booking-Created Flow.md>)
  _EC2 · Node.js · PM2 · Resend · Vercel · payload mapping_

- [PEL-003 — Express Global JSON Middleware Blocks Raw Body Capture in Webhook Routes](<ledger/PEL-003 — Express Global JSON Middleware Blocks Raw Body Capture in Webhook Routes.md>)
  _Express · webhooks · HMAC · raw body · body-parser_

- [PEL-004 — Turbopack JSX Unexpected Token Errors Mislead About Actual Fault Location](<ledger/PEL-004 — Turbopack JSX Unexpected Token Errors Mislead About Actual Fault Location.md>)
  _Next.js · Turbopack · JSX · Vercel build errors_

- [PEL-005 — Vercel ↔ EC2 Payload Field-Name Divergence](<ledger/PEL-005 — Vercel ↔ EC2 Payload Field-Name Divergence.md>)
  _Vercel · EC2 · payload field-name contracts_

- [PEL-006 — pm2 Silence Is Not Proof of Inactivity](<ledger/PEL-006 — pm2 Silence Is Not Proof of Inactivity.md>)
  _PM2 · process diagnostics_

- [PEL-007 — Schema Rename Refactors Require Column-Level Audit](<ledger/PEL-007 — Schema Rename Refactors Require Column-Level Audit.md>)
  _Supabase · schema refactor · column audit_

- [PEL-008 — Silent mcpServers Block Deletion in Claude Desktop Config](<ledger/PEL-008 — Silent mcpServers Block Deletion in Claude Desktop Config.md>)
  _MCP · Claude Desktop config_

- [PEL-009 — Corporate SSL Inspection Silently Breaks Node MCP Attach](<ledger/PEL-009 — Corporate SSL Inspection Silently Breaks Node MCP Attach.md>)
  _MCP · Node · TLS · corporate SSL inspection_

- [PEL-010 — Native ssh.exe MCP Wrappers Fail Silently on Windows](<ledger/PEL-010 — Native ssh.exe MCP Wrappers Fail Silently on Windows.md>)
  _SSH MCP · Windows · ssh2_

- [PEL-011 — Claude's Diagnostic Blind Spots in MCP Environments](<ledger/PEL-011 — Claude's Diagnostic Blind Spots in MCP Environments.md>)
  _MCP · diagnostics · self-correction_

- [PEL-012 — Bridge-Doc Claims About Cross-Surface MCP Availability Need Verification](<ledger/PEL-012 — Bridge-Doc Claims About Cross-Surface MCP Availability Need Verification.md>)
  _MCP · bridge docs · tool_search probing_

- [PEL-013 — Docker-Backed MCP Servers Are Wake-Fragile](<ledger/PEL-013 — Docker-Backed MCP Servers Are Wake-Fragile.md>)
  _MCP · Docker · post-sleep reattach_

- [PEL-014 — Claude Desktop's MCP Running Status Badge Is Not Authoritative](<ledger/PEL-014 — Claude Desktop's MCP Running Status Badge Is Not Authoritative.md>)
  _MCP · status badge · probe-by-tool-call_

- [PEL-015 — PM2 Environment Refresh Requires File Re-Read Not --update-env Flag](<ledger/PEL-015 — PM2 Environment Refresh Requires File Re-Read Not --update-env Flag.md>)
  _PM2 · env-block vars · `pm2 reload <file>`_

- [PEL-016 — Non-Block Env Vars Refresh on Process Respawn Not reload](<ledger/PEL-016 — Non-Block Env Vars Refresh on Process Respawn Not reload.md>)
  _PM2 · non-block env vars · plain `pm2 restart`_

---

## Status

This repository is intentionally minimal — durable troubleshooting knowledge, not discussion. Entries are append-only in spirit: superseded *facts within* an entry are corrected by a newer PEL that references the old one, rather than by silent edits.
