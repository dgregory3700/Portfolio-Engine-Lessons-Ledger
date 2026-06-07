# PEL-002: EC2 Outgoing Email — Booking-Created Flow (Resend via Node.js/PM2)

**Tags:** EC2 · Node.js · PM2 · Resend · Vercel · Next.js · Supabase · Express

---

## 1. Quick Recognition Test

You are trying to:
- Send a confirmation email to a client when a new booking is created
- Send a notification email to the mediator when a new booking is created
- The trigger is a form submission on a Next.js Vercel app
- The email sender lives on EC2 (Node.js/Express/PM2)

Symptoms of this class of problem:
- `Cannot POST /internal/booking-created` — wrong route path
- `{"error":"Unauthorized"}` — missing or wrong internal token header
- `{"error":"Missing client_email or client_name"}` — payload field name mismatch (camelCase vs snake_case)
- `Could not resolve mediator email: User not found` — PM2 running stale in-memory code despite correct file on disk
- Emails never arrive despite `ok: true` response — field mapping silently wrong

---

## 2. Root Cause (Canonical Diagnosis)

This flow has five independent failure points, each silent or misleading:

### A. PM2 serving stale in-memory code
PM2 caches the loaded module in memory. `pm2 restart` alone is not sufficient after file edits — it may reload from cache. The only reliable fix is `pm2 delete all && pm2 start ecosystem.config.cjs`. Confirmed: the error string `"User not found"` can disappear from all files on disk while still appearing in live logs, because PM2 is running an older version of the code in memory.

### B. Wrong route path
The EC2 route is mounted at `/internal` in `server.js` and defined as `/email/booking-created` in `harmonydesk-pro.js`. The full path is `/internal/email/booking-created`. Testing with `/internal/booking-created` returns `Cannot POST`.

### C. Missing internal auth token
The route is protected by `requireInternalToken` middleware, which reads `req.headers["x-internal-token"]` and compares it to `process.env.APP_TOKEN_SECRET`. curl tests and Vercel fetch calls must include this header or receive `401 Unauthorized`.

### D. camelCase vs snake_case payload mismatch
`mapRowToBooking()` in Next.js converts Supabase snake_case columns to camelCase (`client_email` → `clientEmail`). EC2 expects snake_case. Spreading `...booking` from the mapped object silently sends the wrong field names. EC2 validates `booking.client_email` and `booking.client_name` — if these are undefined, it returns `400 Missing client_email or client_name`.

### E. Nested vs flat payload
EC2 does `const booking = req.body` — it expects fields at the top level of the JSON body, not nested under a `booking` key. Sending `{"booking": {...}}` causes the validation check to fail silently.

---

## 3. What NOT to Waste Time On

- **Do not grep for error strings** to confirm whether fixed code is on disk — PM2 may be running a cached version regardless of what's on disk.
- **Do not assume `pm2 restart` flushes module cache** — it does not reliably. Always use `pm2 delete all && pm2 start ecosystem.config.cjs`.
- **Do not spread the mapped Booking object** (camelCase) into the EC2 fetch body. Always construct the EC2 payload manually from raw Supabase `data` or already-parsed variables.
- **Do not nest the payload** under a `booking` key — EC2 reads `req.body` directly.
- **Do not skip the token header** in curl tests — the route will return `401` and you will spend time debugging the wrong thing.

---

## 4. Canonical Fix Pattern

### Step 1 — Confirm the file on disk is correct
```bash
grep -n "mediator_email\|client_email\|User not found" \
  /apps/harmonydesk/harmonydesk-mvp/routes/harmonydesk-pro.js | head -20
```
If `"User not found"` does not appear in the file but does appear in logs → PM2 is stale. Proceed to Step 2.

### Step 2 — Force full PM2 reload
```bash
pm2 delete all && pm2 start ecosystem.config.cjs
```
Verify clean state:
```bash
pm2 show harmonydesk | grep "created at\|restarts"
```
Expect: `restarts = 0`, `created at` = now.

### Step 3 — Confirm the route path
```bash
grep -n "booking-created\|router\|Router" \
  /apps/harmonydesk/harmonydesk-mvp/routes/harmonydesk-pro.js | head -10
grep -n "harmonydesk-pro\|internal" \
  /apps/harmonydesk/harmonydesk-mvp/server.js
```
Full path = mount prefix + route definition. Example: `/internal` + `/email/booking-created` = `/internal/email/booking-created`.

### Step 4 — Get the internal token
```bash
grep "APP_TOKEN_SECRET" /apps/harmonydesk/harmonydesk-mvp/.env
```

### Step 5 — curl test with correct flat payload and token
```bash
curl -X POST http://localhost:3300/internal/email/booking-created \
  -H "Content-Type: application/json" \
  -H "x-internal-token: YOUR_APP_TOKEN_SECRET" \
  -d '{
    "id": "test-123",
    "case_name": "Test Case",
    "client_name": "Test Client",
    "client_email": "you@example.com",
    "mediator_email": "you@example.com",
    "session_start": "2026-04-15T10:00:00.000Z"
  }'
```
Expected response: `{"ok":true,"sent":["client_confirmation","mediator_notification"]}`

### Step 6 — Fix the Next.js fetch payload (snake_case, flat)
In `app/api/bookings/route.ts`, construct the EC2 body manually — never spread the camelCase `booking` object:

```typescript
body: JSON.stringify({
  id: data.id,
  case_name: data.case_name ?? null,
  client_name: clientName,
  client_email: clientEmail,
  client_phone: clientPhone,
  session_start: sessionStart,
  session_end: sessionEnd,
  session_type: sessionType,
  notes,
  mediator_email: mediatorEmail,
}),
```

Where `clientName`, `clientEmail`, etc. are the already-validated variables parsed from `req.body` earlier in the POST handler — not from `mapRowToBooking()`.

---

## 5. Architecture Reference

```
[User submits form]
        ↓
[Next.js /api/bookings POST — Vercel]
        ↓ writes row
[Supabase bookings table]
        ↓ fetch (snake_case flat payload + x-internal-token header)
[EC2 :3300/internal/email/booking-created]
        ↓ requireInternalToken middleware
        ↓ validates client_email, client_name, mediator_email
        ↓ Promise.all
[Resend → client confirmation email]
[Resend → mediator notification email]
```

**EC2 file locations:**
- Server entry: `/apps/harmonydesk/harmonydesk-mvp/server.js`
- Route file: `/apps/harmonydesk/harmonydesk-mvp/routes/harmonydesk-pro.js`
- PM2 config: `/apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs`
- Env file: `/apps/harmonydesk/harmonydesk-mvp/.env`

**Vercel env vars required:**
- `EC2_INTERNAL_URL` — e.g. `http://18.221.48.232:3300`
- `APP_TOKEN_SECRET` — must match EC2 `.env` value exactly

---

## 6. When to Apply This Again

Apply this ledger entry whenever:
- Adding a new EC2 email trigger to any HD Pro route
- A Next.js route POSTs to EC2 and receives `Cannot POST`, `Unauthorized`, or `Missing field` errors
- EC2 logs show errors that do not match any string found in the source files (PM2 stale cache)
- Debugging why emails aren't sending after a code change to `harmonydesk-pro.js`
- Onboarding this pattern to a new SaaS product in the portfolio
