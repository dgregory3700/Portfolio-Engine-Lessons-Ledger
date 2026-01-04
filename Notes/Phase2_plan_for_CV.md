# Phase 2 Handoff — ContentVeritas

**Goal:** Automate post-payment customer experience (provision API access, onboarding).

## ✅ Phase 1 Summary (Done)
- Frontend (Next.js on Vercel) live
- Backend API on EC2 + PM2 + nginx
- Stripe Checkout integrated (Basic/Pro)
- Live mode confirmed (cs_live created)
- Email field editable in live checkout
- PM2 env set with `STRIPE_SECRET_KEY=sk_live_...`

---

## Phase 2 — Post-Payment Automation

### A) Webhook Foundation (Stripe → Server)
- [ ] A1: Add `/webhook/stripe` route with **raw body parser** (before `express.json`)
- [ ] A2: Configure Stripe **TEST** webhook → copy `whsec_test` → set as `STRIPE_WEBHOOK_SECRET`
- [ ] A3: Log events received (`checkout.session.completed`, `customer.subscription.*`, `invoice.payment_failed`)
- [ ] A4: Make TEST purchase → verify event in logs

### B) Provisioning (API Keys + Storage)
- [ ] B1: Add simple store (`keys.db.json` or SQLite) to save `{email, stripe_customer_id, apiKey, plan, status}`
- [ ] B2: On `checkout.session.completed`: create/lookup user, generate API key, set status=active
- [ ] B3: (Optional) Infer plan from price ID; store plan

### C) Onboarding Email
- [ ] C1: Pick email service (SendGrid or SES)
- [ ] C2: Send “Welcome — Your API Key” email on provisioning
- [ ] C3: Include: API key, 30-sec Quickstart (`curl`), docs link, support contact

### D) Protect the API
- [ ] D1: Add middleware to require `Authorization: Bearer <API_KEY>`
- [ ] D2: Validate key, status=active; reject otherwise
- [ ] D3: Add simple `/v1/analyze` guarded endpoint and test

### E) Productionize Webhooks (LIVE)
- [ ] E1: Create **LIVE** webhook endpoint in Stripe → set `STRIPE_WEBHOOK_SECRET=whsec_live_...`
- [ ] E2: Confirm events arrive for a $1 live purchase (refund after)
- [ ] E3: Mark keys **revoked** on `customer.subscription.deleted`
- [ ] E4: Handle `invoice.payment_failed` (warning email + grace period)

### F) Success/Cancel UX polish (customer friendly)
- [ ] F1: `/success` — clear message, “check your email for API key,” quickstart button
- [ ] F2: `/cancel` — no charge taken, button back to Pricing
- [ ] F3: Add “Docs” links where relevant

---

## Quickstart Snippets (for reference)

**Webhook (raw parser)**
```js
app.post('/webhook/stripe', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature'];
  try {
    const event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET);
    // switch(event.type) {...}
    res.status(200).send('[ok]');
  } catch (err) {
    console.error('Webhook signature error:', err?.message || err);
    res.status(400).send('[bad signature]');
  }
});
