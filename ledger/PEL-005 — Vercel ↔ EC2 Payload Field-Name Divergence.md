# PEL-005 — Vercel ↔ EC2 Payload Field-Name Divergence

**Date logged:** May 13, 2026
**Project:** HarmonyDesk Pro
**Related entries:** PEL-002 (same family — Vercel ↔ EC2 payload contract mismatch; PEL-002 was casing divergence, this is field-name divergence)

---

## Symptom

A confirmed mediation session created via the dashboard wrote a row to Supabase but produced no emails — neither the client confirmation nor the mediator notification. `pm2 logs harmonydesk` showed zero new lines on session creation. No errors surfaced in the Vercel function logs at first glance (status was 201 — successful row insert).

## Root cause

The Vercel session-creation API route (`src/app/api/sessions/route.ts`) was firing a `fetch()` to the EC2 email-trigger endpoint with a payload containing `mediator_email` (already resolved on the Vercel side from `supabase.auth.getUser()`). The EC2 handler at `/internal/email/session-confirmed` in `/apps/harmonydesk/harmonydesk-mvp/routes/harmonydesk-pro.js` required `user_id` in the payload so it could perform the mediator email lookup itself via `supabaseAdmin.auth.admin.getUserById(session.user_id)`. The EC2 handler rejected the payload with HTTP 400 and the message `"Missing user_id"`. Vercel's fire-and-forget pattern (try/catch around the fetch with a `console.error`) swallowed the failure silently from the user-visible response.

## Diagnostic technique

The breakthrough came from Vercel's **External APIs** panel in the function invocation detail view (Runtime Logs → expand a /api/sessions POST row). The panel lists every outbound HTTP call made during the function execution, with status codes. Reading bottom-up:

```
POST 18.221.48.232/internal/email/session-confirmed   →   400
```

This single line confirmed three things in one read: (1) the fetch was firing, (2) it was reaching EC2, (3) EC2 was rejecting it. Without the External APIs panel, the diagnosis path would have required adding temporary logging to either side and waiting for another deploy cycle.

## Canonical fix

Vercel handler payload must align with EC2 handler's expected fields. The contract is defined by the EC2 handler — the Vercel side adapts to it, not the other way around.

In this case, the comment in the EC2 handler made the contract explicit: *"Resolve mediator email server-side from user_id (Vercel payload is just the session row)."* The Vercel side was sending a *processed* version (with the email pre-resolved) instead of the *raw row* (with the foreign key). Fix: send `user_id` instead of `mediator_email` and let EC2 do the resolution.

## Operational consequence

When designing Vercel → EC2 internal POSTs, the canonical pattern is:

- **Vercel sends the raw session/case/etc. row (just primary keys + scalar fields).**
- **EC2 resolves any joined or auth-derived data itself using `supabaseAdmin`.**

This pattern keeps EC2 as the source of truth for cross-table lookups and avoids subtle bugs where the same field is resolved on both sides with potentially different logic. It also keeps the Vercel payload smaller and the contract narrower.

## Recurrence prevention

When Vercel calls an EC2 internal route, *always read the EC2 handler's first 10–15 lines* (the validation block) before writing or modifying the Vercel-side fetch body. Mismatch is silent under fire-and-forget patterns. Treat the EC2 handler's required-field checks as the canonical contract definition.

## Related context

This is the second observed instance of "Vercel-EC2 contract mismatch causing silent failure" in HarmonyDesk Pro:

- **PEL-002:** snake_case vs camelCase mismatch on Phase 3 email payloads. Vercel sent `clientEmail`, EC2 expected `client_email`.
- **PEL-005 (this entry):** field-name mismatch. Vercel sent `mediator_email`, EC2 expected `user_id`.

Both are sub-patterns of a broader category: **Vercel ↔ EC2 payload contract divergence**. Future internal-route work should treat the EC2 handler's validation block as canonical and Vercel as the conforming party.
