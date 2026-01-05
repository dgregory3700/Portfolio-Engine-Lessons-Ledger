**EntropyCleaner - Freeze Reason (Identity/Credits mismatch + webhook 500)**
Status: Frozen (not abandoned)
Reason: Runtime identity mismatch caused credits to show as 0 despite DB values; Stripe webhook replay returned 500; verification requires checking auth.users.id vs credits.user_id alignment.
Do not continue until: auth.users.id and credits.user_id are verified to match for the tested email.
