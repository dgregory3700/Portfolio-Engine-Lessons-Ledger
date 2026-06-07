# SaaS Operating System
**Execution Doctrine v1.1 + Session Start Protocol (SSP v1.0)**

This document defines how all SaaS ventures are built, operated, and executed.
It applies across all projects (e.g., HarmonyDesk, ContentVeritas) unless explicitly overridden.

> Changes from v1.0 are marked with `[new in v1.1]` or `[updated in v1.1]`.

---

## I. Operating Environment

- **Vercel-first**
- Production deployments are the test harness
- Local development avoided unless unavoidable
- GitHub is the single source of truth (Web UI or GitHub Desktop preferred)

### Tool Layer `[new in v1.1]`

Three tools now exist in the Claude toolchain. Each has a distinct role and should not substitute for another.

| Tool | Role |
|---|---|
| **Claude.ai chat** | Planning, strategy, copy, architecture decisions. The thinking layer — where all sessions start. |
| **Claude in Chrome** | Browser automation, live inspection of deployed pages, contact scraping, form interactions. |
| **Claude Code** | Writing and editing code directly inside a repo. Runs in terminal (Termius / local). Reads files, commits to GitHub. |

**Default rule:** Chat plans it. Claude Code builds it. Chrome verifies it live. Don't use one tool for another's job.

---

## II. Code Handling Rules

- Prefer **full-file replacement** over partial diffs
- Do not edit or invent file contents without seeing the file
- The assistant must **ask to see files** before making changes
- No speculative folder structures, routes, or schemas

### Claude Code reads files directly `[new in v1.1]`

When a session is designated Claude Code mode, file upload to chat is not required — Claude Code reads the repo directly. For chat-only sessions, the original rule applies: share the file in chat first.

---

## III. Validation Loop `[updated in v1.1]`

0. **Plan in Claude.ai chat** — define the change clearly before touching code `[new]`
1. Write or update file (via Claude Code or directly)
2. Commit to GitHub
3. Deploy via Vercel
4. Inspect deployment logs
4b. **Optional:** use Claude in Chrome to inspect live page behavior `[new]`
5. Iterate based on real runtime behavior

Local simulations are not authoritative.

---

## IV. SaaS Factory Pipeline

1. Idea validation
2. Domain purchase
3. GitHub repository
4. Vercel deployment
5. Supabase
6. SendGrid or Resend
7. Stripe or LemonSqueezy
8. Public launch
9. Post-launch hardening

---

## V. Scope Philosophy

- Shipping beats polishing
- Production truth beats theory
- Deletion beats complexity
- Observability beats elegance

---

## VI. Session Start Protocol (SSP) `[updated in v1.1]`

Every work session must begin with this block.

### SSP Template

```
Context:        <Venture Name>
Mode:           <Primary Mode> (+ <Secondary Mode>)
Tools:          <Chat only | Chat + Chrome | Chat → Claude Code>
Objective:      <Concrete outcome for this session>
Time Horizon:   <Duration or deadline>

Last Session:   <One sentence — what was completed>
Current State:  <One sentence — where things stand right now>
Blocking Risks: <Any known issues or unknowns that could block progress>

Definition of Done:
  - ...
  - ...

Files to Load:
  - <filename> — reason needed
  - <filename> — reason needed

Constraints:    (optional)
Memory:         <Do not retain | Retain if stable | Update venture memory>
```

### SSP Rules

- **One venture per session**
- **One objective per session**
- Mode must be explicitly declared
- Tools line declares which Claude tools are active for this session
- "Done" must be binary-verifiable
- Constraints override default behavior
- Memory is opt-in and intentional
- It is the user's responsibility to paste the SSP, handoff doc, and any listed files at the start of each new session

---

## VI-B. Session Close + Handoff Protocol `[new in v1.1]`

When a session's Definition of Done is met, Claude generates three artifacts before closing:

### 1. Pre-filled SSP for next session
A completed SSP template ready to paste into the new chat. `Last Session`, `Current State`, and `Files to Load` are filled in based on what was accomplished.

### 2. Handoff doc
A brief document covering:
- **What was done** — summary of completed work
- **Current state** — exact state of the codebase, deployment, or campaign
- **What's next** — the logical next objective
- **Open risks** — anything unresolved, untested, or potentially fragile
- **Decisions made** — key choices made during the session and why

### 3. File list
An explicit list of files to paste into the next session, with a one-line reason for each. Only files that are actually needed — no padding.

### Handoff rule
Claude cannot carry memory across sessions on its own. The handoff doc is the bridge. The more complete it is, the faster the next session reaches productive work.

---

## VII. Authority

Execution Doctrine governs **how** work is done.
SSP governs **what** is done in this session.

If a conflict exists:
> **Execution Doctrine wins unless SSP explicitly overrides it.**

---

## VIII. Change Control

To change either doctrine or SSP defaults:
- State the change explicitly
- Declare it doctrine- or SSP-level
- Confirm whether memory should be updated

Until changed, this system governs all SaaS work.

---

*Execution Doctrine v1.1 — updated to include Claude toolchain and Session Close + Handoff Protocol*
