# 🔐 PEL-001  
## Stripe Webhook Returns 500 Due to Ambiguous Supabase RPC Function

---

### Ledger ID
**PEL-001**

### Status
**Resolved — Canonical Fix Established**

---

## 1. Quick Recognition Test (Read This First)

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

column reference "user_id" is ambiguous

sql
Copy code

However, when surfaced through **PostgREST** (Supabase’s RPC layer), this error was **sometimes misreported** as:

- a schema cache error, or  
- a missing function error  

This obscured the true cause and led to wasted debugging effort.

This was **not** a Stripe issue.  
This was **not** a permissions issue.  
This was **not** a webhook routing issue.

It was a **SQL function design flaw when used as an RPC contract**.

---

## 3. What NOT to Waste Time On

When this issue appears, **do not** spend time on:

- Reconfiguring Stripe Checkout  
- Regenerating webhook secrets  
- Rewriting webhook routing logic  
- Reinstalling Supabase  
- Rechecking RLS policies repeatedly  
- Adding fallback parameter names  
- Repeatedly reloading PostgREST schema *without changing the function*

These actions do **not** address the root cause.

---

## 4. Canonical Fix Pattern (AI-Oriented Section)

**Do not attempt to repair the original function. Bypass it entirely.**

### Required Fix Strategy

Create a **new RPC function** with:

- A **new name** (e.g. `_v2`)
- `RETURNS void`
- Unique parameter names (`p_user_id`, `p_add`, etc.)
- Fully qualified table references
- Explicit `GRANT EXECUTE`
- PostgREST schema reload

### Canonical SQL Pattern

```sql
create or replace function public.add_user_credits_v2(
  p_user_id uuid,
  p_add integer
)
returns void
language plpgsql
security definer
set search_path = public
as $$
begin
  insert into public.user_credits (user_id, credits_remaining, updated_at)
  values (p_user_id, p_add, now())
  on conflict (user_id)
  do update set
    credits_remaining = public.user_credits.credits_remaining + excluded.credits_remaining,
    updated_at = now();
end;
$$;

grant execute on function public.add_user_credits_v2(uuid, integer) to service_role;
grant execute on function public.add_user_credits_v2(uuid, integer) to authenticated;
grant execute on function public.add_user_credits_v2(uuid, integer) to anon;

select pg_notify('pgrst', 'reload schema');
Application Code Rules
Call only the new _v2 function

Do not expect return data

Treat the RPC as a side-effect operation

Remove all legacy parameter names (e.g. p_amount)

5. When to Apply This Again
Apply this lesson any time:

Stripe webhooks mutate state via Supabase RPC

RPC calls return unexplained HTTP 500 errors

Errors reference schema cache inconsistently

A function returns a table but is used only for side effects

Column and parameter naming overlaps exist

This pattern will recur in:

Credit systems

Subscription entitlements

Usage metering

Feature gating

Quota enforcement

6. Resolution Confirmation
Stripe webhook returns HTTP 200

Credits increment atomically

Dashboard reflects updated balance

No Stripe retries

Issue permanently resolved via v2 RPC

