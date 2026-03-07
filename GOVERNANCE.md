# SaaS Operating System  
**Execution Doctrine v1.0 + Session Start Protocol (SSP v1.0)**

This document defines how all SaaS ventures are built, operated, and executed.  
It applies across all projects (e.g., HarmonyDesk, ContentVeritas) unless explicitly overridden.

---

# I. Execution Doctrine (Global, Persistent)

## 1. Operating Environment
- **Vercel-first**
- Production deployments are the test harness
- Local development avoided unless unavoidable
- GitHub is the single source of truth (Web UI or GitHub Desktop preferred)

## 2. Code Handling Rules
- Prefer **full-file replacement** over partial diffs
- Do not edit or invent file contents without seeing the file
- The assistant must **ask to see files** before making changes
- No speculative folder structures, routes, or schemas

## 3. Validation Loop
1. Write or update file  
2. Commit to GitHub  
3. Deploy via Vercel  
4. Inspect deployment logs  
5. Iterate based on real runtime behavior  

Local simulations are not authoritative.

## 4. SaaS Factory Pipeline
1. Idea validation  
2. Domain purchase  
3. GitHub repository  
4. Vercel deployment  
5. Supabase  
6. SendGrid or Resend
7. Stripe or LemonSqueezy  
8. Public launch  
9. Post-launch hardening  

## 5. Scope Philosophy
- Shipping beats polishing  
- Production truth beats theory  
- Deletion beats complexity  
- Observability beats elegance  

---

# II. Session Start Protocol (SSP)

Every work session must begin with this block.

## SSP Template

Context: <Venture Name>

Mode: <Primary Mode> (+ <Secondary Mode>)

Objective: <Concrete outcome>

Time Horizon: <Duration or deadline>

Definition of Done:

…

…

…

Constraints:

… (optional)

Memory: <Do not retain | Retain if stable | Update venture memory>


---

## SSP Rules

- **One venture per session**
- **One objective per session**
- Mode must be explicitly declared
- “Done” must be binary-verifiable
- Constraints override default behavior
- Memory is opt-in and intentional

---

# III. Authority

Execution Doctrine governs **how** work is done.  
SSP governs **what** is done in this session.

If a conflict exists:
> **Execution Doctrine wins unless SSP explicitly overrides it.**

---

# IV. Change Control

To change either:
- State the change explicitly  
- Declare it doctrine- or SSP-level  
- Confirm whether memory should be updated  

Until changed, this system governs all SaaS work.


