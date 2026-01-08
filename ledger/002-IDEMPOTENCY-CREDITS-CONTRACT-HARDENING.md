# Ledger ID: PE-LEDGER-IDEMPOTENCY-CREDITS-CONTRACT-HARDENING
## Title: Credit System Idempotency Failure from Unstable Credits Contract

---

## Quick Recognition Text
Webhook or API credit logic appears correct, but Vercel build fails or runtime behavior is unsafe due to inconsistent credit field naming and optional values.

---

## Root Cause (Canonical Diagnosis)
The credit system relied on a shared `CreditsRow` object whose shape was not stable across the system.

Specifically:
- Some Supabase RPCs returned `balance`
- API routes and guards expected `credits_remaining`
- The shared credits library exposed optional or ambiguous fields
- TypeScript correctly blocked deployment
- Runtime idempotency guarantees were undermined by contract ambiguity

This was a **credit system hardening failure**, not a Stripe or Supabase failure.

---

## What NOT to Waste Time On
- Adding optional chaining or fallbacks in API routes
- Patching individual webhook handlers
- Weakening TypeScript types
- Treating idempotency as “only a database concern”
- Debugging Stripe retries when the contract is unstable

---

## Canonical Fix Pattern (AI-Oriented Section)
**Stabilize the credit contract at the library boundary.**

The shared credits module must:
- Export **one canonical credit field**
- Guarantee it always exists
- Normalize backend variance internally
- Enforce idempotency through atomic RPCs

### Canonical `lib/credits.ts`
```ts
import { createClient } from "@/lib/supabase/server";

export type CreditsRow = {
  user_id: string;
  credits_remaining: number;
  updated_at?: string;
};

function normalizeCreditsRow(row: any): CreditsRow {
  const credits =
    typeof row?.credits_remaining === "number"
      ? row.credits_remaining
      : typeof row?.balance === "number"
        ? row.balance
        : 0;

  return {
    user_id: row.user_id,
    credits_remaining: credits,
    updated_at: row.updated_at,
  };
}

export async function getOrCreateCredits(userId: string): Promise<CreditsRow> {
  const supabase = await createClient();
  const { data, error } = await supabase.rpc("get_or_create_user_credits", {
    p_user_id: userId,
  });
  if (error) throw new Error(error.message);
  return normalizeCreditsRow(Array.isArray(data) ? data[0] : data);
}

export async function addCreditsAtomic(
  userId: string,
  amount: number
): Promise<CreditsRow> {
  const supabase = await createClient();
  const { data, error } = await supabase.rpc("add_user_credits", {
    p_user_id: userId,
    p_amount: amount,
  });
  if (error) throw new Error(error.message);
  return normalizeCreditsRow(Array.isArray(data) ? data[0] : data);
}

export async function decrementOneCreditAtomic(
  userId: string
): Promise<CreditsRow> {
  const supabase = await createClient();
  const { data, error } = await supabase.rpc("decrement_one_credit", {
    p_user_id: userId,
  });
  if (error) throw new Error(error.message);
  return normalizeCreditsRow(Array.isArray(data) ? data[0] : data);
}

export async function applyStripeCreditPackIdempotent(input: {
  stripeEventId: string;
  eventType: string;
  sessionId: string | null;
  userId: string;
  creditsToAdd: number;
}): Promise<{ processed: boolean }> {
  try {
    await addCreditsAtomic(input.userId, input.creditsToAdd);
    return { processed: true };
  } catch (e: any) {
    const msg = String(e?.message || "").toLowerCase();
    if (msg.includes("duplicate") || msg.includes("unique") || msg.includes("idempot")) {
      return { processed: false };
    }
    throw e;
  }
}
