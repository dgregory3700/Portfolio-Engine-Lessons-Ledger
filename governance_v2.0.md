# GOVERNANCE.md  
**SaaS Operating System — Execution Doctrine v1.0**

This repository operates under a standardized execution system designed to ensure consistency, velocity, and production-grade outcomes across all SaaS ventures.

This document is binding for all contributors (human or AI).

---

## 1. Governing Principles

This repository is governed by four primary principles:

- **Production over theory**  
- **Shipping over polishing**  
- **Observability over elegance**  
- **Deterministic execution over experimentation**

All decisions must optimize for real-world deployment and revenue-generating operation.

---

## 2. Operating Environment

### 2.1 Deployment Model
- **Vercel-first**
- Production deployments are the canonical test environment
- Local development is avoided unless technically unavoidable

### 2.2 Source Control
- GitHub is the single source of truth
- GitHub Web UI and GitHub Desktop are preferred interfaces

---

## 3. Code Change Policy

### 3.1 File Authority
- Changes are made via **full-file replacement**, not inline edits
- No file may be modified without being explicitly provided

### 3.2 No-Guessing Rule
Contributors and assistants must not:
- Assume directory structures
- Invent files
- Guess schemas, routes, or dependencies

Files must be requested and reviewed before changes are proposed.

---

## 4. Validation Protocol

All changes follow the same validation loop:

1. Update file(s)
2. Commit to GitHub
3. Deploy via Vercel
4. Inspect deployment logs
5. Iterate based on observed behavior

Local builds and simulations are not authoritative.

---

## 5. SaaS Factory Pipeline

Every venture follows this standardized lifecycle:

1. Idea validation  
2. Domain acquisition  
3. GitHub repository creation  
4. Vercel deployment  
5. Supabase integration  
6. SendGrid integration  
7. Stripe or LemonSqueezy integration  
8. Public launch  
9. Post-launch hardening  

Deviation from this pipeline requires explicit justification.

---

## 6. External Services Policy

All third-party services must:
- Use official SDKs or APIs
- Be production-grade
- Provide logs or dashboards

Primary services include:
- Supabase  
- SendGrid  
- Stripe or LemonSqueezy  
- Vercel  

---

## 7. Collaboration Contract (AI & Human)

All collaborators must:
- Request files before modifying them
- Provide full-file outputs when making changes
- Optimize for Vercel deployment success
- Use production logs as primary debugging evidence

Speculative refactors, hypothetical solutions, and local-only workflows are discouraged.

---

## 8. Scope & Release Discipline

- Features that delay launch are deferred
- Broken but observable beats perfect but invisible
- Removal is preferred over complexity

The objective is to get **real users on real infrastructure paying real money** as quickly and safely as possible.

---

## 9. Authority & Changes

This governance system applies unless explicitly overridden.

To change any rule:
- The change must be stated explicitly
- It must be declared a governance-level modification
- Memory retention must be confirmed

Until amended, this document governs all activity in this repository.
