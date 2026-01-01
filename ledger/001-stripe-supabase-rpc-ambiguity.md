# 🔐 PEL-001  
## Stripe Webhook Returns 500 Due to Ambiguous Supabase RPC Function

---

### Ledger ID
**PEL-001**

### Status
**Resolved — Canonical Fix Established**

---

## 1. Quick Recognition Text (Read This First)

You are very likely encountering **this exact issue** if:

- Stripe Checkout completes successfully  
- Stripe webhook is delivered but returns **HTTP 500**  
- Stripe retries the webhook  
- Error messages include one or more of:
  - `Could not find the function ... in the schema cache`
  - `column reference "user_id" is ambiguous`
- Supabase permissions appear correct  
- Webhook signature verification passes  
- Credits, entitlements, or usage counters do **not** update  

If these symptoms match → **this is the issue**.

---

## 2. Root Cause (Canonical Diagnosis)

The original Supabase RPC function (`add_user_credits`) was **structurally unsafe for RPC usage**.

Specifically:

- The function returned a table  
  (e.g. `RETURNS TABLE(user_id, credits_remaining)`)
- Output column names overlapped with:
  - function parameters
  - table column names
- Internal SQL referenced identifiers such as `user_id` without qualification

PostgreSQL correctly raised:

```text
column reference "user_id" is ambiguous
