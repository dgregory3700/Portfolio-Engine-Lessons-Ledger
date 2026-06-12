# DuelTech AI Architecture v1 Blueprint

## Overview

DuelTech is an independent software studio building focused SaaS products, including HarmonyDesk, Content Veritas, TextPolish, and Entropy Cleaner.[1] This blueprint defines a practical v1 operating system for DuelTech: a human-supervised, AI-assisted company architecture designed to reduce founder bottlenecks by turning repeated work into structured workflows, specialized agent roles, and controlled execution layers.[1][2]

The purpose of v1 is not to simulate a fully autonomous startup. The purpose is to create a durable operating model that helps a solo founder move from fragmented manual execution toward supervised AI-assisted departments that can research, prepare, route, draft, inspect, and report work across the business.[1][3]

## Context and correction

Earlier discussions centered heavily on HarmonyDesk and Entropy Cleaner because the initial operating summary emphasized those two products. That emphasis should not be interpreted as the full scope of DuelTech. DuelTech publicly presents itself as a focused SaaS studio with multiple products, including HarmonyDesk, Content Veritas, TextPolish, and Entropy Cleaner, so this architecture is written for the whole venture system rather than only two projects.[1]

This distinction matters because the architecture should not be tied to one product’s current stage. It should support a portfolio model in which products move through discovery, validation, build, launch, commercialization, maintenance, and retirement using the same core operating rules.[1]

## Architecture principles

The v1 system is built around six principles:

- Human-supervised execution, not blind autonomy.[2][4]
- Narrow workflows before broad platforms.[1]
- One primary objective per session or operating cycle.[1]
- Least-privilege access to tools and accounts.[2][4]
- Evidence-backed recommendations and reports.[3][2]
- Reusable systems across multiple products.[1]

These principles match DuelTech’s public product philosophy of focused execution and also align with the emerging Perplexity direction toward tool-connected, multi-step task handling across files, tools, and complex workstreams.[1][3][2]

## System model

DuelTech AI Architecture v1 has five layers.

| Layer | Purpose | v1 responsibility |
|---|---|---|
| Command layer | Founder control surface | Define objectives, launch sessions, approve actions, review reports |
| Memory layer | Business context | Store doctrines, SOPs, product states, tool map, prompts, decisions |
| Department layer | Specialized operating units | Perform scoped research, drafting, planning, inspection, and coordination |
| Execution layer | Real-world action mechanisms | Use Zapier, Perplexity Computer, scripts, APIs, and connected apps to carry out approved work |
| Approval and audit layer | Safety and traceability | Gate high-risk actions, preserve logs, document what changed |

Perplexity Pro is positioned around advanced models and access to Perplexity Computer for complex tasks, while Perplexity Enterprise is positioned as a secure platform for orchestrating work across files and tools.[5][2] That means v1 should be designed so it works immediately with Pro, but can later scale into a more connector-heavy and security-governed model if DuelTech upgrades to Enterprise.[5][2][4]

## Command layer

The command layer is the founder-facing operating surface. It is where the business is directed, where sessions are launched, and where all significant actions originate.

Recommended v1 command functions:

- Intake a business objective.
- Assign a primary department owner.
- Define the product, metric, deadline, and risk class.
- Record constraints and required approvals.
- Produce a report at the end of the cycle.

### Command template

```text
DuelTech Command
Objective:
Department owner:
Product:
Primary metric:
Systems involved:
Risk level:
Definition of done:
Constraints:
Needs approval for:
Deadline:
```

### Report template

```text
DuelTech Report
Objective:
What was inspected:
What changed:
What worked:
What failed:
Required decision:
Next recommended action:
Evidence:
```

This layer should live in one place, such as a Notion workspace, internal markdown repo, or structured operating dashboard, so that direction, status, and decisions do not fragment across disconnected chats.[3][2]

## Memory layer

The memory layer is the persistent business context that lets departments work with continuity. It should capture stable knowledge that would otherwise have to be re-explained every session.

Core v1 memory categories:

- Operating doctrines and rules.
- Product records for HarmonyDesk, Content Veritas, TextPolish, Entropy Cleaner, and future products.[1]
- ICP notes and validation findings.
- SOPs for launch, QA, billing, outreach, onboarding, and support.
- Tool inventory and account map.
- Prompt templates such as SSPs.
- Decision history and rejected ideas.

The memory layer should separate timeless records from active-session notes. Timeless records hold doctrine and product definitions; active notes hold current hypotheses, blockers, tasks, and weekly operating conclusions.

## Department layer

The department layer is the core of the architecture. Each department has a narrow role, standard inputs, standard outputs, and clear boundaries.

### 1. Executive Operations

Purpose:
- Convert business goals into structured sessions.
- Route work to the correct department.
- Track status, blockers, and approvals.
- Produce weekly decision memos.

Typical outputs:
- SSPs.
- Priority memos.
- Cross-department status summaries.
- Weekly operating review.

### 2. Validation Department

Purpose:
- Pressure-test opportunities.
- Identify buyers, competitors, and evidence.
- Recommend Advance, Pause, or Reject decisions.

Typical outputs:
- Lead validation memos.
- Competitor scans.
- ICP evidence summaries.
- Risk findings.

### 3. Product Department

Purpose:
- Translate validated opportunities into scoped products.
- Define workflows, user journeys, feature exclusions, and product promises.
- Ensure marketing claims match real functionality.

Typical outputs:
- Product briefs.
- Scope definitions.
- User workflow maps.
- Offer and narrative frameworks.

### 4. Engineering Department

Purpose:
- Inspect repositories.
- Package implementation work.
- Produce build plans, code-change requests, QA checklists, and release readiness reviews.

Typical outputs:
- Engineering task plans.
- Pull-request specifications.
- QA checklists.
- Defect summaries.

### 5. Infrastructure Department

Purpose:
- Inspect and manage environment configuration, deployment surfaces, backend services, billing rails, DNS, and operational integrations.
- Verify production readiness and reduce silent failures.

Typical outputs:
- Deployment checklists.
- Environment audits.
- Integration status reports.
- Risk logs.

### 6. Revenue and Marketing Operations

Purpose:
- Own positioning, outreach systems, content production, lead routing, nurture logic, and conversion reporting.
- Reduce founder dependence on ad hoc marketing effort.

Typical outputs:
- Messaging maps.
- Campaign drafts.
- Follow-up sequences.
- Weekly funnel reports.
- Content plans.

### 7. Customer Operations

Purpose:
- Support onboarding, support intake, churn signal review, documentation updates, and customer issue summaries.
- Turn customer friction into operational insight.

Typical outputs:
- Onboarding improvements.
- FAQ and documentation updates.
- Support summaries.
- Churn-risk notes.

## Product portfolio model

This blueprint applies to the entire DuelTech portfolio, not just one active build. DuelTech publicly identifies multiple focused SaaS products, including HarmonyDesk, Content Veritas, TextPolish, and Entropy Cleaner.[1]

Each product should live in one of the following operating states:

| State | Meaning | Primary department |
|---|---|---|
| Discovery | Problem exploration, early hypotheses | Validation |
| Validation | Evidence gathering and decision-making | Validation |
| Definition | Scope, workflow, and offer design | Product |
| Build | Implementation and integration work | Engineering + Infrastructure |
| Launch | Public release, surface QA, and messaging | Revenue + Infrastructure |
| Growth | Funnel tuning, content, outreach, retention | Revenue + Customer Ops |
| Maintenance | Stability, support, incremental updates | Engineering + Customer Ops |
| Retired / Parked | Not active, but preserved | Executive Ops |

A product record should always store its current state, owner department, top constraint, top metric, and next required decision.

## Execution layer

The execution layer is how plans become real-world actions. In v1, this layer should be simple and modular rather than overly autonomous.

Recommended execution components:

- Zapier for cross-app triggers and workflow glue.
- Perplexity Pro for research, synthesis, drafting, and structured reasoning.[5]
- Perplexity Computer for future multi-step execution across connected tools when enabled.[3]
- GitHub and Claude/Copilot for code-side implementation support.
- Email and comms tools such as Instantly, Twilio, SendGrid, Brevo, and related systems where relevant.
- Product infrastructure tools such as Vercel, Supabase, Stripe, and related backend services.

Perplexity Computer is described as a general-purpose digital worker that can handle multi-step work across the web, files, and connected tools.[3] That makes it a strong candidate for future execution tasks such as document routing, research preparation, admin work, and low-risk operational actions after approvals are defined.[3]

## Approval and audit layer

The architecture should use three risk classes.

| Risk class | Examples | Approval rule |
|---|---|---|
| Low | Research, drafting, summarization, tagging, internal notes | Auto-allowed |
| Medium | Draft campaigns, test sends, issue creation, staging updates, sandbox actions | Auto-allowed with logging or light approval |
| High | Production deploys, billing changes, live outbound, DNS edits, database writes, customer-impacting automations | Founder approval required |

This least-privilege model aligns with Perplexity Enterprise’s emphasis on secure orchestration and governance controls for business use.[2][4]

## Tool permission model

Each connected system should be mapped by department and permission scope.

| System | Department(s) | Default access | Approval required for |
|---|---|---|---|
| GitHub | Engineering | Read, issue creation | Merge, direct production edits |
| Vercel | Infrastructure | Read logs, inspect project status | Deploys, env var edits |
| Supabase | Infrastructure | Read schema and inspect data model | Migrations, destructive writes |
| Stripe | Infrastructure | Read products and customer state | Live billing changes |
| Resend / SendGrid / Brevo | Revenue, Infrastructure | Drafts, test sends | Production sequences |
| Instantly | Revenue | Campaign drafting and reporting | Live sends |
| Twilio | Revenue, Customer Ops | Sandbox mode | Live messaging |
| Zapier | Executive Ops, Infrastructure | Draft workflows | Live customer-facing automations |
| Cal.com | Revenue, Customer Ops | Schedule routing | Public booking logic changes |
| Bouncer | Revenue | Verification access | No elevated permissions beyond workflow use |

A simple tool registry should document the account owner, credential location, risk class, rollback path, and whether the system is read-only, draft-capable, or write-enabled.

## Standard workflows

DuelTech AI Architecture v1 should standardize a small number of repeatable workflows.

### Workflow 1: Idea to decision

- Input: market opportunity, pain point, or product hypothesis.
- Owner: Validation.
- Output: Advance, Pause, or Reject memo.

### Workflow 2: Validated lead to scoped product

- Input: approved opportunity.
- Owner: Product.
- Output: scoped product brief, exclusions, workflow map, initial promise.

### Workflow 3: Scope to launch-ready build

- Input: product brief.
- Owner: Engineering + Infrastructure.
- Output: deployed, verified product with checklist evidence.

### Workflow 4: Lead to meeting or signup

- Input: target segment and offer.
- Owner: Revenue.
- Output: campaign, responses, booked calls, or activated signups.

### Workflow 5: Support signal to product improvement

- Input: support issue, friction note, churn risk, FAQ gap.
- Owner: Customer Ops + Product.
- Output: doc update, fix recommendation, onboarding improvement, or escalation.

### Workflow 6: Weekly operating review

- Input: department reports, blockers, metrics, pending approvals.
- Owner: Executive Ops.
- Output: next-week priorities and decisions.

## Weekly operating cadence

A lightweight cadence prevents drift while preserving founder bandwidth.

### Monday
- Executive Ops creates the weekly operating brief.
- Top priorities and blocked approvals are set.

### Midweek
- Department reports are updated.
- High-risk actions wait for review.
- Bottlenecks are summarized.

### Friday
- Executive Ops generates a weekly memo.
- Decisions are recorded.
- Products move state only when evidence supports the move.

This creates a rhythm in which departments do scoped work and the founder remains the final decision-maker rather than the person manually doing every operational step.[1]

## Recommended v1 implementation stack

A practical v1 stack should favor simplicity.

| Need | Recommended v1 choice |
|---|---|
| Command hub | Notion, markdown repo, or lightweight internal dashboard |
| Knowledge base | Markdown docs + structured product records |
| Workflow glue | Zapier |
| Research and drafting | Perplexity Pro |
| Future multi-step execution | Perplexity Computer when enabled[3] |
| Code execution support | Claude + GitHub + Copilot |
| Email operations | Instantly + Brevo or SendGrid |
| Scheduling | Cal.com |
| CRM substitute | Airtable, Notion database, or structured sheet |
| Incident and QA logging | Markdown docs or Notion database |

The strongest v1 rule is this: if a system does not save time, reduce mistakes, or improve decisions, it should not be added.

## Implementation roadmap

### Phase 1: Foundation

- Create the command hub.
- Create product records for every active and inactive product.[1]
- Record doctrines, SSP templates, risk rules, and tool registry.
- Define department owners and default outputs.

### Phase 2: Workflow core

- Implement idea-to-decision workflow.
- Implement build-to-launch checklist flow.
- Implement weekly operating review.
- Add a simple department reporting template.

### Phase 3: Execution and automations

- Connect Zapier.
- Build low-risk automation flows first.
- Add reporting feeds from core systems.
- Keep all high-risk actions behind approval gates.

### Phase 4: Revenue and customer loops

- Add Revenue workflows for outreach, content, and funnel summaries.
- Add Customer Ops workflows for onboarding, support, and docs.
- Feed recurring objections and support pain back into Product and Validation.

### Phase 5: Computer-enabled operations

- When Perplexity Computer is enabled, test it with low-risk, reversible tasks first.[5][3]
- Expand only after observing reliability, auditability, and control.

## Anti-patterns to avoid

The architecture should explicitly avoid these failure modes:

- One giant “AI assistant” with no departmental boundaries.
- Unlogged automations touching production systems.
- Tool sprawl without a permission registry.
- Marketing treated as ad hoc activity rather than an operating function.
- Product state tracked in memory alone instead of in a shared system.
- High-risk actions executed without founder approval.
- Building agent complexity before workflow clarity.

## Definition of success for v1

DuelTech AI Architecture v1 is successful if it produces the following outcomes:

- Fewer context switches for the founder.
- Faster Advance, Pause, or Reject decisions.
- Clearer product records across the portfolio.[1]
- More repeatable build, launch, and review cycles.
- Reduced operational drift between planning and execution.
- A reusable system that supports multiple focused SaaS products, not just a single app.[1]

The goal is not to imitate a large company. The goal is to compress the work of several startup functions into a disciplined, auditable, AI-assisted operating model directed by one founder.[1][3]
