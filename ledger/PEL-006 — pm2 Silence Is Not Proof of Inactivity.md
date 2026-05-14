# PEL-006 — pm2 Silence Is Not Proof of Inactivity

**Date logged:** May 13, 2026
**Project:** HarmonyDesk Pro
**Related entries:** PEL-005 (the bug PEL-006 was discovered during)

---

## Symptom

During Cat 2B email pipeline diagnosis, `pm2 logs harmonydesk --lines 30` was used as a real-time diagnostic — open the tail, create a fresh test session in the dashboard, expect log output. Tail printed nothing. The natural inference was "the request never reached EC2." This inference turned out to be incomplete.

The request *was* reaching EC2 in some cases (returning 400, rejected by validation) and *was* reaching EC2 successfully in other cases (returning 200 after the fix shipped). pm2 was silent in both scenarios.

## Root cause

The EC2 Express app does not have request-logging middleware (no `morgan` or equivalent). Express by default does not log incoming requests. pm2's log stream only surfaces what the application explicitly writes to stdout/stderr via `console.log` / `console.error` (or equivalent).

The `/email/session-confirmed` handler in `harmonydesk-pro.js` had:

- `console.error` on validation failures (400 paths)
- `console.error` on the catch block (500 path)
- **No log on the success path** (200) — handler returned `res.json({ ok: true, ... })` silently

The 400 path *did* print to pm2 in some cases (e.g., the `console.error` for "Could not resolve mediator email"). But the specific 400 returned in this case was the early-return `Missing user_id` check, which had no log. So that 400 was silent too.

Net effect: pm2 was silent regardless of whether the handler succeeded, failed early, or succeeded with no surfaced output.

## Diagnostic technique

The fix was to look at Vercel's **External APIs** panel (PEL-005's diagnostic technique) instead of relying on pm2. Vercel's logs surfaced the 400 status that pm2 hid. This bypassed the EC2-side observability gap entirely.

After the email pipeline fix shipped, a follow-up change added a success-path log to the handler:

```js
console.log(`[session-confirmed] sent for ${session.id} client=${session.client_email || "—"} mediator=${mediatorEmail}`);
```

This single line restored pm2 as a valid observability channel for this route. The next test session printed the line in real time, confirming the handler ran end-to-end.

## Canonical fix

Every internal EC2 route that doesn't have a logging middleware *must* have at least one `console.log` on its success path. The log line should include enough fields to verify the handler ran correctly without needing to consult another tool:

- Primary identifier (session id, case id, document id, etc.)
- Key resolved values (email addresses, file names, counts)
- Any meaningful path branches (e.g., "client confirmation skipped: no client_email")

Format suggestion: `[<route-tag>] <action> for <id> <key>=<value> <key>=<value>`.

## Operational consequence

`pm2 tail` silence after a triggering action means one of three things:

1. The request never reached EC2.
2. The request reached EC2 but failed before any logging code path.
3. **The request reached EC2 and succeeded but the handler never logs success.**

In a debugging session, **rule out (3) first** by reading the handler code — it's a one-minute check that prevents an hour of chasing (1) and (2).

## Recurrence prevention

Convention going forward:

- **New internal route:** always include a success-path `console.log` before the success return. This is part of the route handler's definition of done, not an afterthought.
- **Existing internal route with no success log:** add one whenever the route is touched for any reason. Don't wait for an outage to discover the observability gap.
- **Diagnosing a pm2-silence symptom:** before trusting the silence, `grep` the route handler for `console.log` and `console.error` in the success block. If absent, the silence is meaningless — pivot to Vercel's External APIs panel or add temporary logging.

## Related context

This is an observability hygiene issue, distinct from the contract/payload bugs of PEL-002 and PEL-005, but discovered during the same incident. The lesson generalizes beyond HarmonyDesk Pro: any service in the DuelTech stack reached via fire-and-forget from another service needs explicit success-path logging to be operationally diagnosable.
