# Advisor Access Enablement Checklist

**Purpose:** Provide a simple, repeatable procedure when a mediator
trainer or advisor agrees to join HarmonyDesk and provide feedback.

------------------------------------------------------------------------

## When to Use This

Use this checklist **after** you receive an email reply from a potential
advisor who agrees to participate.

This process grants them temporary access **without payment**, while
keeping the Stripe provisioning system intact.

------------------------------------------------------------------------

## Step 1 --- Create User in Supabase Auth

1.  Go to **Supabase Dashboard**

2.  Navigate to:

    Authentication → Users

3.  Click:

    **Add user**

4.  Enter:

    Email: advisor@example.com\
    Password: temporary password (any secure value)

5.  Click:

    **Create user**

------------------------------------------------------------------------

## Step 2 --- Add Subscription Row (Trial Access)

Go to:

Supabase → Table Editor → `subscriptions`

Click:

**Insert row**

Fill in:

-   user_email: advisor@example.com
-   status: trialing
-   stripe_customer_id: NULL
-   stripe_subscription_id: NULL
-   trial_end_at: NULL
-   current_period_end_at: NULL
-   price_id: NULL

Then click:

**Save**

Important rule:

The email must match the Auth email exactly.

------------------------------------------------------------------------

## Step 3 --- Confirm Login Works

Ask the advisor to:

1.  Visit:

    https://app.harmonydesk.ai/login

2.  Log in using:

    Their email\
    The temporary password

If they can access the dashboard, access provisioning is complete.

------------------------------------------------------------------------

## Step 4 --- Send Welcome / Advisory Email

Send a simple confirmation message such as:

Subject: Welcome to HarmonyDesk --- Advisor Access Enabled

Message:

Your access is now active.

You can log in here: https://app.harmonydesk.ai/login

There is no billing attached to your account.

We appreciate your feedback and suggestions.

------------------------------------------------------------------------

## Step 5 --- Later Conversion to Paid (Automatic)

If the advisor later purchases using the same email:

No manual action is required.

The system will:

-   Update the existing subscription row
-   Attach Stripe IDs
-   Change status to active
-   Preserve all data

------------------------------------------------------------------------

## Troubleshooting

### Login fails

Check:

-   Email spelling matches exactly
-   Subscription status is "trialing"
-   User exists in Auth

### Duplicate subscription rows

Do not create a second row.

Always reuse the same email.

------------------------------------------------------------------------

## Summary

This process safely grants advisory access while preserving:

-   Billing integrity
-   Subscription lifecycle tracking
-   Future upgrade compatibility
-   Data continuity
