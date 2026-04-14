# Cal.com Integration — HarmonyDesk Pro Setup Guide

Use this guide to connect a mediator's Cal.com account to their HarmonyDesk Pro dashboard. Once connected, any booking made through Cal.com will automatically create a session in HarmonyDesk Pro and send confirmation emails to both the mediator and the client.

---

## Prerequisites

- Active HarmonyDesk Pro subscription
- A Cal.com account (free tier works)

---

## Step 1 — Find your HarmonyDesk Webhook URL

1. Log in to your HarmonyDesk Pro dashboard at `pro.harmonydesk.ai`
2. Go to **Settings → Integrations**
3. Under the Cal.com section, copy the webhook URL shown in the **Your Cal.com webhook URL** field

It will look like this:
```
https://api.harmonydesk.ai/internal/webhooks/calcom/YOUR-USER-ID
```

Keep this tab open — you will need this URL in the next step.

---

## Step 2 — Create the Webhook in Cal.com

1. Log in to your Cal.com account
2. Go to `app.cal.com/settings/developer/webhooks`
3. Click **+ New**
4. Fill in the form:
   - **Subscriber URL:** paste the URL you copied from Step 1
   - **Enable webhook:** toggle ON
   - **Event triggers:** click **Clear all triggers**, then add **Booking created** only
   - **Secret:** type any password you want — you will need to remember this (example: `MySecret2026!`)
   - **Custom Payload Template:** leave OFF
5. Click **Ping test** — the Webhook response should show `passed` and `Status: 200`
6. If the ping test passes, click **Create webhook**

---

## Step 3 — Save the Secret in HarmonyDesk Pro

1. Go back to your HarmonyDesk Pro dashboard
2. Go to **Settings → Integrations**
3. In the **Cal.com webhook secret** field, type the same secret you entered in Cal.com
4. Click **Save & connect**
5. The status badge should change to **Connected**

---

## Step 4 — Verify It Works

1. Go to your Cal.com booking page and make a test booking using a real email address you can check
2. Within a few seconds, check:
   - A new session appears in your HarmonyDesk Pro **Sessions** page
   - The client receives a confirmation email
   - You (the mediator) receive a notification email

---

## Troubleshooting

**Ping test fails with 404**
The subscriber URL is wrong. Make sure it includes `/internal/` in the path:
`https://api.harmonydesk.ai/internal/webhooks/calcom/YOUR-USER-ID`

**Ping test fails with 401**
You have not saved your secret in HarmonyDesk yet. Complete Step 3 first, then ping test again.

**Ping test passes but booking does not appear in HarmonyDesk**
The secret you saved in HarmonyDesk does not match the secret in Cal.com. Go to Settings → Integrations, re-enter your secret, and click Update secret.

**Cal.com rejects the subscriber URL**
Cal.com requires `https://`. Make sure the URL starts with `https://` not `http://`.

---

## Notes for Mediators

- You only need to do this setup once per Cal.com account
- If you ever change your Cal.com webhook secret, update it in HarmonyDesk Settings → Integrations immediately or bookings will stop syncing
- Cal.com sessions and manually-added sessions coexist in the same Sessions list — there is no conflict
- Client phone numbers are not collected by Cal.com by default — you can add them manually to a session after it is created if needed

---

*Last updated: April 2026 — HarmonyDesk Pro Phase 3b*
