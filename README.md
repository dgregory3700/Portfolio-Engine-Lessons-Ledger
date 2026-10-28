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
| `ledger/` | **PEL-000 → PEL-041** — atomic, reusable troubleshooting lessons. The core of this repo. |
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

- [PEL-017 — Block Env Vars Refresh on pm2 reload of the Ecosystem File and Show in pm2 env](<ledger/PEL-017 — Block Env Vars Refresh on pm2 reload of the Ecosystem File and Show in pm2 env.md>)

- [PEL-018 — PM2 Non-Block Env Vars Refresh Only on a Full Recreate, Not Restart or reload](<ledger/PEL-018 — PM2 Non-Block Env Vars Refresh Only on a Full Recreate, Not Restart or reload.md>)

- [PEL-019 — Disabling Legacy Supabase Keys Deactivates anon and service_role Together](<ledger/PEL-019 — Disabling Legacy Supabase Keys Deactivates anon and service_role Together.md>)

- [PEL-020 — Dead-Var Verdicts Must Be Derived on the Consuming Surface, Not the Surface It Points At](<ledger/PEL-020 — Dead-Var Verdicts Must Be Derived on the Consuming Surface, Not the Surface It Points At.md>)

- [PEL-021 — Node MCP cmd Shim Name Can Differ From Package Name; Resolve the Real Binary](<ledger/PEL-021 — Node MCP cmd Shim Name Can Differ From Package Name; Resolve the Real Binary.md>)

- [PEL-022 — force-dynamic on a Next.js Layout Does Not Re-Run It on Soft Sibling Nav; a Layout-Sourced Value Needs a Client Poll](<ledger/PEL-022 — force-dynamic on a Next.js Layout Does Not Re-Run It on Soft Sibling Nav; a Layout-Sourced Value Needs a Client Poll.md>)

- [PEL-023 — A Price-Gate Cutover Is Safe Only If the Write Path Emits the Gated Literal; Trace the price_id Writer, Don't Trust the Plan](<ledger/PEL-023 — A Price-Gate Cutover Is Safe Only If the Write Path Emits the Gated Literal; Trace the price_id Writer, Don't Trust the Plan.md>)

- [PEL-024 — Enabling RLS on Live Tables; Mirror an Already-RLS-On Sibling's Policy and Confirm the Public Route's Client Is Service-Role First](<ledger/PEL-024 — Enabling RLS on Live Tables; Mirror an Already-RLS-On Sibling's Policy and Confirm the Public Route's Client Is Service-Role First.md>)

- [PEL-025 — Probe Gate Predicates Against Candidate Sources Empirically Before Spending Run Budget](<ledger/PEL-025 — Probe Gate Predicates Against Candidate Sources Empirically Before Spending Run Budget.md>)

- [PEL-026 — Source Probes Must Replay the Engine's Real Request; Some Hosts Block Automated Clients Outright](<ledger/PEL-026 — Source Probes Must Replay the Engine's Real Request; Some Hosts Block Automated Clients Outright.md>)
  _probing · user-agent · request headers · regulations.gov · eCFR · govinfo · reginfo_

- [PEL-027 — Measure, Don't Assume, Before Slating a Source](<ledger/PEL-027 — Measure, Don't Assume, Before Slating a Source.md>)
  _probing · fetch timeout · char windows · extractor offsets · drift markers_

- [PEL-028 — Diag-Before-Run Is the Standing Pre-Run Instrument; Replay the Full Pipeline and Print Criterion Traces](<ledger/PEL-028 — Diag-Before-Run Is the Standing Pre-Run Instrument; Replay the Full Pipeline and Print Criterion Traces.md>)
  _diag replay · admission gate · run budget · run isolation · OE_MAX_SOURCE_URLS_

- [PEL-029 — Preserve Every Source URL at Read Time; Recover From Exports, Never Reconstruct](<ledger/PEL-029 — Preserve Every Source URL at Read Time; Recover From Exports, Never Reconstruct.md>)
  _record keeping · URLs · probes · diag reads_

- [PEL-030 — SSH MCP Transport Limits and Error Semantics; Verify State Before Any Retry](<ledger/PEL-030 — SSH MCP Transport Limits and Error Semantics; Verify State Before Any Retry.md>)
  _SSH MCP · timeouts · exit codes · detached jobs · bridge drops_

- [PEL-031 — Moving Code and Content Over SSH Safely; Quoting Failure Classes and the Routes That Work](<ledger/PEL-031 — Moving Code and Content Over SSH Safely; Quoting Failure Classes and the Routes That Work.md>)
  _SSH MCP · bash -lc quoting · heredoc · base64 · patchers · scratch hygiene_

- [PEL-032 — pm2 stop Without pm2 save Is Not a Durable Stop](<ledger/PEL-032 — pm2 stop Without pm2 save Is Not a Durable Stop.md>)
  _PM2 · systemd · resurrect · dump file_

- [PEL-033 — Ledgers Are the Source of Truth for Sequential IDs and Pending Work; Handoff Notes Go Stale](<ledger/PEL-033 — Ledgers Are the Source of Truth for Sequential IDs and Pending Work; Handoff Notes Go Stale.md>)
  _D-numbers · PEL numbers · OE build-status numbers · handoff staleness_

- [PEL-034 — reginfo and OMB Data Surfaces; Use the Bulk XML and Supporting Statements, Not Scraped Pages](<ledger/PEL-034 — reginfo and OMB Data Surfaces; Use the Bulk XML and Supporting Statements, Not Scraped Pages.md>)
  _reginfo · PRAXML · ICR inventory · Supporting Statements · docx_

- [PEL-035 — regulations.gov v4 API Search Semantics Are Not What They Look Like](<ledger/PEL-035 — regulations.gov v4 API Search Semantics Are Not What They Look Like.md>)
  _regulations.gov · API · search · comments · rate limits_

- [PEL-036 — Evidence-Class Ceilings; Know What a Source Type Can and Cannot Prove Before Hunting It](<ledger/PEL-036 — Evidence-Class Ceilings; Know What a Source Type Can and Cannot Prove Before Hunting It.md>)
  _evidence quality · buyer evidence · WTP · pain evidence · price patterns_

- [PEL-037 — Terrain Screening; Run the Cheapest Kill First and Expect Value to Attract Incumbents](<ledger/PEL-037 — Terrain Screening; Run the Cheapest Kill First and Expect Value to Attract Incumbents.md>)
  _terrain selection · incumbents · stage zero · screening order_

- [PEL-038 — HTML Region Extraction Pitfalls; Regex Region Pickers Truncate Nested and Card Layouts](<ledger/PEL-038 — HTML Region Extraction Pitfalls; Regex Region Pickers Truncate Nested and Card Layouts.md>)
  _HTML extraction · region picker · regex · vendor pages_

- [PEL-039 — Harness and Replay Discipline; Fixtures From Measured Text, and Know Which Changes Can Be Replayed](<ledger/PEL-039 — Harness and Replay Discipline; Fixtures From Measured Text, and Know Which Changes Can Be Replayed.md>)
  _test harness · fixtures · consequence replay · rank sentinels_

- [PEL-040 — Gated-Run Runner Safety; Restore the Gate on Any Exit and Verify From the Log](<ledger/PEL-040 — Gated-Run Runner Safety; Restore the Gate on Any Exit and Verify From the Log.md>)
  _gate · atomic open/run/close · md5 · run budget · run reports_

- [PEL-041 — Read the Ratified Text Before a Doctrine Arc; and a Second Device Gets Its Own SSH Key](<ledger/PEL-041 — Read the Ratified Text Before a Doctrine Arc; and a Second Device Gets Its Own SSH Key.md>)
  _doctrine drift · ledger reading · SSH keys · multi-device_

---

## Status

This repository is intentionally minimal — durable troubleshooting knowledge, not discussion. Entries are append-only in spirit: superseded *facts within* an entry are corrected by a newer PEL that references the old one, rather than by silent edits.
