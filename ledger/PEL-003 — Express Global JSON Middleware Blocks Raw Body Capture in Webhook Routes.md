# PEL-003 — Express Global JSON Middleware Blocks Raw Body Capture in Webhook Routes

**Tags:** Express · Node.js · EC2 · PM2 · Cal.com · Webhooks · HMAC · Supabase · body-parser

---

## 1. Quick Recognition Test

- A webhook route uses `express.raw({ type: "application/json" })` to capture raw body for HMAC signature verification
- Ping tests to the webhook URL return 200 and log correctly
- Real event webhooks (e.g. `BOOKING_CREATED`) fire successfully from the third-party service (confirmed by UI confirmation page, emails from that service, etc.)
- EC2 out log shows **only** ping responses — no booking event, no signature error, no insert attempt
- Supabase table remains empty after confirmed real bookings
- No errors in EC2 log at all

---

## 2. Root Cause

`express.json()` is registered globally in `server.js` **before** the router is mounted. It processes every incoming request body, consuming the readable stream. By the time the route-level `express.raw()` middleware runs, the body stream is already exhausted — `req.body` is a parsed JS object, not a Buffer.

`Buffer.isBuffer(req.body)` returns `false`, so the handler silently falls into the ping branch and returns `200 { ok: true, ping: true }` — with no log entry, no error, and no further processing.

The ping test passes because Cal.com sends an empty body for pings. `Buffer.isBuffer(emptyBuffer)` also returns false, which accidentally produces the correct ping response — masking the bug entirely during initial testing.

**The specific pattern that causes this:**

```js
// server.js
app.use(express.json({
  verify: (req, _res, buf) => {
    req.rawBody = buf; // saves buffer here...
  },
}));

app.use("/internal", hdProRoutes); // ...but route uses express.raw(), too late
```

```js
// routes/harmonydesk-pro.js (broken)
router.post("/webhooks/calcom/:userId", express.raw({ type: "application/json" }), async (req, res) => {
  const rawBody = req.body; // req.body is already a parsed object, not a Buffer
  if (!Buffer.isBuffer(rawBody) || rawBody.length === 0) {
    // always hits this branch for real bookings — returns ping response silently
  }
});
```

---

## 3. What NOT to Waste Time On

- **Do not re-check the Cal.com webhook URL.** If ping tests are landing and logging, the URL is correct.
- **Do not re-check the webhook secret / HMAC logic.** The signature check is never reached — the handler bails before it gets there.
- **Do not re-check Supabase RLS or insert logic.** The insert is never attempted.
- **Do not suspect Cal.com is not firing.** If the booking confirmation page loads and the third-party sends its own confirmation email, the webhook fired.
- **Do not restart PM2 or redeploy hoping it clears.** This is a code logic issue, not a process state issue.
- **Do not add more logging inside the HMAC block.** Logs will never appear there until the body capture is fixed.

---

## 4. Canonical Fix Pattern

The global `express.json()` `verify` callback already captures the raw body into `req.rawBody`. Use that instead of route-level `express.raw()`.

**Two changes to the webhook route:**

1. Remove `express.raw({ type: "application/json" })` from the route arguments
2. Replace `req.body` with `req.rawBody` for the raw buffer

```js
// BEFORE (broken)
router.post("/webhooks/calcom/:userId", express.raw({ type: "application/json" }), async (req, res) => {
  const rawBody = req.body;
  if (!Buffer.isBuffer(rawBody) || rawBody.length === 0) { ... }
  const expectedSig = crypto.createHmac("sha256", secret).update(rawBody).digest("hex");
  const event = JSON.parse(rawBody.toString());
```

```js
// AFTER (fixed)
router.post("/webhooks/calcom/:userId", async (req, res) => {
  const rawBody = req.rawBody;
  if (!rawBody || rawBody.length === 0) { ... }
  const expectedSig = crypto.createHmac("sha256", secret).update(rawBody).digest("hex");
  const event = JSON.parse(rawBody.toString());
```

**Prerequisite:** `server.js` must have the `verify` callback on the global `express.json()` call:

```js
app.use(express.json({
  verify: (req, _res, buf) => {
    req.rawBody = buf;
  },
}));
```

If it does not, add it. This is the single correct pattern for raw body access in an Express app that uses global JSON parsing.

---

## 5. When to Apply This Again

Apply this entry whenever **all** of the following are true:

- Express app uses global `express.json()` or `bodyParser.json()` registered before route handlers
- A route needs the **raw unparsed body** for HMAC/signature verification (Stripe, Cal.com, GitHub, Twilio, etc.)
- Route-level `express.raw()` is being used to try to capture that raw body
- Webhook ping tests pass but real event payloads produce no log output and no side effects

This pattern applies to **any** HMAC-verified webhook in this stack — Stripe, Cal.com, GitHub webhooks, Twilio, or any future integration requiring signature verification against a raw request body.
