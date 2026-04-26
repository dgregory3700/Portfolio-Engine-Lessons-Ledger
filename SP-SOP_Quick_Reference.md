# SP-SOP — Quick Reference

## Purpose

SP-SOP (Stripe Provisioning Standard Operating Pipeline) is the standard onboarding architecture used to convert a paying customer into an authenticated application user in a deterministic, auditable, production-safe manner.

This document is a condensed operational reference. It defines the minimum required behavior for implementing the onboarding pipeline across all SaaS products.

---

# Core Principle

Payment creates entitlement.

User access is granted only after controlled provisioning.

Never create users directly from the frontend.

---

# Canonical Flow

Landing Page
↓
Stripe Checkout
↓
Stripe Webhook
↓
Idempotency Ledger
↓
Subscription Provisioning
↓
Password Setup Token
↓
Welcome Email
↓
Password Creation
↓
Authenticated Login

---

# Required System Components

## Stripe Checkout

Collect payment and create the subscription.

Must include user email in metadata.

Checkout does not create users.

---

## Webhook Receiver

Receives Stripe events and starts provisioning.

Must:

* Verify Stripe signature
* Log all events
* Process idempotently

Stripe is the source of truth.

---

## Idempotency Ledger

Stores webhook processing state.

Purpose:

* Prevent duplicate execution
* Enable safe retries
* Provide audit history

---

## Subscription Record

The subscription row controls application access.

Required fields:

user_email
status
stripe_customer_id
stripe_subscription_id

One row per user email.

---

## Password Setup Token

Provides controlled user creation.

Rules:

* Token must be hashed
* Token must expire
* Token must be single-use

Standard expiration: 24 hours.

---

## Email Delivery

Send onboarding instructions after provisioning succeeds.

Email failure must be logged and visible.

No silent failures.

---

## Password Creation Endpoint

Validates token and creates the authentication user.

Must verify:

* Token validity
* Token expiration
* Subscription eligibility

---

# Access Gating Rule

Access is allowed only when:

subscription.status is one of:

trialing
active

All other states deny access.

Authorization must be enforced on the server.

---

# Failure Handling Requirements

The system must be:

Deterministic
Observable
Recoverable

Examples:

Duplicate webhook → ignored safely
Expired token → rejected
Email failure → logged
Webhook retry → safe

---

# Logging Requirements

Log the following events:

Webhook received
Webhook processed
Subscription updated
Token generated
Email sent
Password created
Login granted
Access denied

---

# Operational Definition of Done

The onboarding pipeline is considered working when the following sequence completes successfully:

Payment completed
Webhook received
Event recorded
Subscription updated
Token generated
Email delivered
Password created
User login successful
Access granted

---

# Portfolio Rule

All future SaaS products must implement SP-SOP unless a documented exception exists.

SP-SOP is infrastructure, not product logic.

Reuse it without modification whenever possible.
