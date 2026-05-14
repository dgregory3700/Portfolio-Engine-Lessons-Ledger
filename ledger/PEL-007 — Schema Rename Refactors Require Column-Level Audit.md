# PEL-007 — Schema Rename Refactors Require Column-Level Audit

**Date logged:** May 13, 2026
**Project:** HarmonyDesk Pro
**Related entries:** None directly — this is a new pattern.

---

## Symptom

After Cat 2B Part 1 consolidated the `bookings` and `sessions` tables into a unified `sessions` table, the `/api/cases` route surfaced a Vercel runtime error on every invocation:

```
Supabase GET /api/cases (sessions lookup) error:
{ code: '42703', details: null, hint: null, message: 'column sessions.date does not exist' }
```

The error did not break the UI visibly — the route had a fail-soft branch that degraded `nextSessionDate` to null. So no user complaint surfaced. The error was discovered incidentally while diagnosing the Cat 2B email pipeline issue (PEL-005), surfaced by widening the Vercel log view to show 30 minutes of activity rather than narrowly filtering on a single endpoint.

## Root cause

When the unified `sessions` table was introduced, the Cat 2B Part 1 migration:

- Renamed the data source from `bookings` to `sessions` everywhere it appeared in queries (good — `from("sessions")` was used)
- Did not audit the column references *within* those queries against the new schema

The `/api/cases` route queried `sessions` with three column references inherited from the `bookings` schema:

| Reference used | Existed on `bookings`? | Exists on unified `sessions`? |
|---|---|---|
| `user_email` | yes | no — replaced by `user_id` |
| `completed` (boolean) | yes | no — replaced by `status` (enum) |
| `date` | yes | no — renamed to `start_time` |

Of these, only `date` triggered a visible 42703 error because it was the column Supabase parsed first in the query. The other two would have failed in turn if `date` had succeeded — but the early error masked the rest.

## Diagnostic technique

The error message is unambiguous (`column sessions.date does not exist`) and points directly at the file. Once located, a defensive read of the full query block — not just the highlighted column — was the key step. Fixing only the highlighted column would have left two more failures lurking.

## Canonical fix

When refactoring a table rename, the audit checklist is:

1. **Grep for the old table name across the entire codebase.** Replace with new name. This is the obvious step.
2. **For each replaced query, audit the column references against the new schema.** Compare each column name in `.select()`, `.eq()`, `.gt()`, `.lt()`, `.in()`, `.is()`, `.order()`, `.insert()`, `.update()` against the new table's column list. Stale references will compile fine in TypeScript (Supabase types are often `any`-loose) and only fail at runtime.
3. **Audit any related auth helpers.** If the old schema used `user_email` and the new uses `user_id`, any local auth helper that returns only `userEmail` will need to also expose `userId`. This was the case in `/api/cases` — `requireAuthedSupabase()` was a locally-defined helper returning `userEmail` only.

## Operational consequence

Table-rename migrations create a deceptive sense of completeness. The compile step passes. The smoke test passes (if the smoke test doesn't exercise the renamed columns). The bug surfaces only when an actual query hits a renamed column, which can be days or weeks after the migration shipped if the route is low-traffic or has a fail-soft degradation path.

The HD Pro `/api/cases` route fell into this gap because:

- The fail-soft degradation hid the user-visible impact
- The route was low enough traffic that the error was buried in Vercel logs unless explicitly searched for
- The original Cat 2B Part 1 verification focused on the *positive* path (calendar populates correctly) rather than the *negative* path (do any other routes still reference the old schema?)

## Recurrence prevention

After every schema migration (rename, drop, or column-type change), add an explicit step to the migration checklist:

- **Grep for every changed column name across all `src/app/api/**/*.ts` files.**
- **Grep for every changed column name across the EC2 routes directory.**
- **For each hit, verify the column exists on the new schema with the same name and type.** Note: this includes column references in queries that don't directly select the column (`.eq()`, `.gt()`, `.order()`, etc. — Supabase will error on any column reference, not just `.select()`).
- **Run the Vercel runtime logs filter at "Last 1 hour" with no path filter** after deploying and triggering a representative cross-section of routes. Scan for any `42703` (column does not exist) or `42P01` (relation does not exist) errors. These two codes are the canonical signals of a missed reference.

## Related context

Cat 2B Part 1 (table consolidation) was completed in a prior session and verified primarily by the calendar UI rendering correctly. That verification proved the new schema was *queryable in the most common path* but not that every route had been updated to the new column shape. PEL-007 formalizes the lesson: rename migrations need a dedicated "stale reference" sweep, not just a "happy path" smoke test.
