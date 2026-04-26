# SP-SOP — Stripe Provisioning Standard Operating Pipeline

## Purpose

SP-SOP (Stripe Provisioning Standard Operating Pipeline) is the canonical onboarding architecture used to convert a paying customer into an authenticated application user in a deterministic, auditable, production-safe manner.

This document defines the **standard operating model** for all current and future SaaS products built within the portfolio. It is intended to be reused across repositories as a reference implementation and as an operational contract for both human operators and automated agents.

The objective is not convenience. The objective is **reliability, traceability, and repeatability**.

SP-SOP ensures that:

* Payment confirmation is authoritative
* Provisioning is idempotent
* User creation is controlled
* Access gating is deterministic
* Failures are observable and recoverable

This pipeline has been validated in production and is considered stable infrastructure.

---

# Core Principle

**Payment does not create a user.**
**Payment creates entitlement.**
**User access is granted only after controlled provisioning.**

This separation prevents:

* orphaned accounts
* duplicate users
* inconsistent subscription states
* unauthorized dashboard access
* silent failures

---

# System Overview

## Canonical Flow

Landing Page
↓
Stripe Checkout
↓
Stripe Webhook
↓
Idempotency Ledger
↓
Provisioning Logic
↓
Password Setup Token
↓
Welcome Email
↓
Password Creation
↓
Authenticated Login

---

# Architectural Components

## 1) Stripe Checkout

### Role

Collect payment and initiate the onboarding lifecycle.

### Responsibilities

* Collect billing information
* Create subscription
* Trigger webhook event
* Provide authoritative payment confirmation

### Requirements

* Stripe-hosted checkout session
* Metadata must include:

user_email
product_identifier
(optional) referral or campaign data

### Design Rule

Checkout does not provision users.

It only signals eligibility for provisioning.

---

## 2) Stripe Webhook Receiver

### Role

The webhook is the entry point into the provisioning system.

It converts an external payment event into an internal provisioning workflow.

### Hosting Pattern

Typically hosted on:

Node.js (Express)
PM2-managed service
Dedicated infrastructure (e.g., EC2)

### Required Event Types

checkout.session.completed
customer.subscription.created
customer.subscription.updated
customer.subscription.deleted

### Mandatory Behavior

* Verify Stripe signature using raw request body
* Reject invalid signatures
* Log all events
* Process events idempotently

### Design Rule

The webhook must never trust client-side state.

Stripe is the single source of truth.

---

## 3) Idempotency Ledger

### Role

Prevent duplicate processing of webhook events.

### Table

stripe_webhook_events

### Required Fields

id
stripe_event_id
status
attempt_count
created_at
processed_at

### Status Values

received
processing
completed
failed

### Mandatory Behavior

* Record every event
* Detect duplicates
* Retry safely
* Provide audit history

### Design Rule

Every webhook event must be safe to process multiple times.

---

## 4) Provisioning Engine

### Role

Create or update the subscription record that governs access to the application.

### Table

subscriptions

### Required Fields

user_email
status
stripe_customer_id
stripe_subscription_id
trial_end_at
current_period_end_at
price_id

### Allowed Status Values

trialing
active
past_due
canceled
incomplete

### Mandatory Behavior

* Upsert subscription record
* Normalize Stripe lifecycle states
* Maintain a single row per user_email

### Design Rule

Subscription state is the access gate.

---

## 5) Password Setup Token System

### Role

Provide controlled user creation without exposing credentials or requiring manual account creation.

### Table

password_setup_tokens

### Required Fields

token_hash
user_email
expires_at
used
created_at

### Security Requirements

* Token stored as SHA-256 hash
* Token expiration enforced
* Token single-use

### Standard Expiration

24 hours

### Mandatory Behavior

* Generate token after successful provisioning
* Reuse active token when appropriate
* Invalidate token after use

### Design Rule

User creation must be intentional and verifiable.

---

## 6) Email Delivery System

### Role

Deliver onboarding instructions to the user.

### Supported Providers

Resend (preferred)
SendGrid (legacy fallback)

### Email Content Requirements

* Password setup link
* Expiration notice
* Support contact information

### Mandatory Behavior

* Send email only after provisioning success
* Log delivery attempts
* Surface failures explicitly

### Design Rule

No silent email failures.

---

## 7) Password Creation Endpoint

### Route

/api/set-password

### Role

Convert a valid token into a real authenticated user account.

### Mandatory Behavior

* Validate token hash
* Check expiration
* Confirm subscription eligibility
* Create authentication user
* Mark token as used

### Failure Conditions

Token expired
Token already used
Subscription not eligible
Invalid token

### Design Rule

Authentication must depend on subscription state.

---

## 8) Login System

### Role

Provide controlled dashboard access.

### Access Rule

User access is granted only if subscription status is:

trialing
active

### Enforcement Location

Server-side authentication middleware

### Design Rule

Authorization must be server-enforced.

---

# Canonical Access Gating Rule

Access is allowed only when:

subscription.status IN ("trialing", "active")

All other states deny access.

---

# Failure Handling Model

## Required Properties

Deterministic
Observable
Recoverable

## Examples

Duplicate webhook

Ignored safely

Expired token

Rejected explicitly

Email failure

Logged and surfaced

Webhook retry

Processed idempotently

---

# Security Model

## Core Protections

Webhook signature verification
Token hashing
Single-use tokens
Subscription-based access control
Server-side authorization

## Non-Negotiable Rules

Never trust client state
Never create users from the frontend
Never bypass subscription gating
Never allow silent provisioning failures

---

# Logging Requirements

Every stage must produce logs.

## Required Log Events

Webhook received
Webhook processed
Subscription updated
Token generated
Email sent
Password created
Login granted
Access denied

---

# Observability Requirements

The system must allow operators to answer the following questions immediately:

Did Stripe send the event?
Was the webhook processed?
Was the subscription updated?
Was the token generated?
Was the email delivered?
Did the user create a password?
Is the subscription eligible?

If any answer is unknown, observability is insufficient.

---

# Reusability Contract

SP-SOP is not tied to a specific product.

It is a reusable infrastructure pattern.

## Intended Usage

New SaaS launch
Vertical SaaS clone
Product pivot
Multi-product portfolio

## Expected Stability

High

## Expected Modification Frequency

Low

Most changes should occur in:

UI
Features
Domain logic

Not in:

Onboarding pipeline

---

# Deployment Model

Typical production configuration:

Frontend

Next.js application
Hosted on Vercel

Webhook Service

Node.js Express server
Hosted on dedicated infrastructure
Managed by PM2

Database

Supabase

Email Provider

Resend

Payment Processor

Stripe

---

# Versioning Strategy

This document defines the canonical onboarding architecture.

Changes to this pipeline must be versioned explicitly.

## Recommended Version Tag

SP-SOP v1

Future revisions should increment:

v2
v3
v4

Never modify behavior silently.

---

# Operational Definition of Done

The onboarding pipeline is considered production-ready when the following sequence succeeds end-to-end:

User completes payment
Webhook receives event
Event recorded in ledger
Subscription row created or updated
Password token generated
Email delivered
User sets password
User logs into dashboard
Access granted

---

# Portfolio Rule

All future SaaS products must implement SP-SOP unless a documented exception exists.

Deviation from this architecture requires explicit justification.

---

# Summary

SP-SOP is the standard onboarding engine for the portfolio.

It provides:

Deterministic provisioning
Subscription-based access control
Secure user creation
Auditable lifecycle tracking
Production-grade reliability

This system is considered foundational infrastructure and should be reused without modification whenever possible.
