# Ledger ID: PE-LEDGER-ENTROPY-VERCEL-TS-CREDITS-CONTRACT
## Title: TypeScript Build Failure from Unstable Credits Contract Across API Routes

---

## Quick Recognition Text
Vercel build fails with TypeScript errors stating that `credits_remaining` does not exist or is possibly `undefined` when accessed in API routes.

---

## Root Cause (Canonical Diagnosis)
The `CreditsRow` type exported from `lib/credits.ts` did not match the expectations of multiple API routes.  
Specifically:
- Some routes required `credits.credits_remaining` to exist and be a number.
- The shared credits module exposed optional or mismatched fields (`balance` vs `credits_remaining`).
- TypeScript correctly flagged this as unsafe during build, causing Vercel compilation failure.

The failure was caused by an **unstable data contract** between the credits library and its consumers.

---

## What NOT to Waste Time On
- Patching individual API routes with optional chaining or fallback logic.
- Adding conditional checks (`??`, `||`, `if (!credits)`) in multiple call sites.
- Suppressing TypeScript errors or weakening types.
- Debugging Supabase RPC behavior beyond confirmed return shapes.

---

## Canonical Fix Pattern (AI-Oriented Section)
Stabilize the contract at the source.

Enforce a **single canonical shape** for credits returned from `lib/credits.ts`:
- `credits_remaining` must always exist.
- It must always be a number.
- Any backend variance (`balance` vs `credits_remaining`) must be normalized internally.

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
