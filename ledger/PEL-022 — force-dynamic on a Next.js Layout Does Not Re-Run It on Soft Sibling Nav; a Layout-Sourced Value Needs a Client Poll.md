# PEL-022 — `force-dynamic` on a Next.js Layout Does Not Re-Run It on Soft Sibling Navigation; a Layout-Sourced Value Needs a Client Poll, Not a Route-Segment Flag

**Date logged:** June 20, 2026
**Project:** HarmonyDesk Pro — Cat UI-FR (session-request freshness fix); first Cat shipped under the AUTONOMOUS dial.
**Related entries:** None direct — this is the first Next.js App Router caching entry in the ledger. Conceptually adjacent to PEL-020 (reason about the surface that actually *consumes* the value, not the one it appears to live on): here the badge value is consumed by a client component but *sourced* in a layout, and the fix must live where it's consumed.

---

## Symptom / context

On `pro.harmonydesk.ai`, a new public session request (submitted via the unauthenticated `/request/[slug]` page) did not appear on the mediator's dashboard on **soft** (client-side) navigation. Neither the `Session Requests` nav-badge count nor the requests list updated until a hard refresh. Two surfaces were stale, and they had two different root causes — only one of which `force-dynamic` actually fixes.

- The **list** at `(dashboard)/sessions/requests/page.tsx` is a server component with no route-segment config; the Router Cache could serve a stale RSC payload on soft nav.
- The **badge** count is computed in `(dashboard)/layout.tsx` (`getRequestedSessionCount(user.id)`) and passed to the client `DashboardShell` as the `requestCount` prop.

## Root cause / mechanism

Two separate caching behaviors, easy to conflate:

1. **The list (server component):** `export const dynamic = "force-dynamic"` is the correct fix. It forces a fresh server render on every request, so soft-nav into the page re-fetches and shows new rows. This is a page/segment re-render — straightforward.

2. **The badge (layout-sourced value):** the trap. **Next.js layouts do NOT re-execute on client-side soft navigation between sibling pages.** A layout runs on full page load (and on `router.refresh()`), then *persists* across soft navigations within its segment — that persistence is the whole point of layouts. So any value the layout computes once and passes down as a prop is **frozen** for the life of that layout instance, regardless of route-segment config. Critically, **`force-dynamic` on the layout does NOT fix this** — a dynamic layout is still not re-run on soft sibling nav; `dynamic` controls render-time caching of *that segment's output*, not whether the layout function re-invokes on client navigation. The badge therefore stayed at its full-load value until a hard refresh, no matter what flag was added to the layout.

## Diagnostic technique / verification

- Reproduce on **soft nav only** (click within the app), never a hard refresh — a hard refresh re-runs the layout and masks the bug entirely. If "it works after F5 but not when clicking around," suspect a layout-sourced value, not a fetch/cache flag.
- Confirm the value's *source*: grep for where the prop is computed. If it's in a `layout.tsx` and passed down, no page-level or layout-level `dynamic`/`revalidate` flag will refresh it on soft nav.
- Note what already works: in this app, approve/decline *did* decrement the badge correctly, because `RequestsInboxClient` calls `router.refresh()` on success — and `router.refresh()` **does** re-run the layout. That asymmetry (works on in-app action, fails on idle/soft-nav arrival) is the tell: the only paths that refresh a layout are full load and `router.refresh()`.
- Verified fixed on prod: submitted a live request while idle on the dashboard; badge incremented within the poll interval with no hard refresh; the list showed the row; approve still decremented and fired the Cat 2H emails.

## Canonical fix / pattern

A layout-sourced value that must stay fresh on soft nav has to **update itself client-side**. The shipped pattern:

1. **List half:** `export const dynamic = "force-dynamic"` on the server-component page. (This part the flag *does* fix.)
2. **Badge half:** make the value self-updating in the client component.
   - New cookie-bound, RLS-aware endpoint `GET /api/requests/count` that **mirrors the layout query exactly** (same `.eq` filters), returns `{ count }`. No service-role.
   - In `DashboardShell` (already a client component): seed local state from the `requestCount` prop (no first-paint flash), then poll the endpoint on **mount + `window` focus + `visibilitychange` + a fixed interval (45s here)**. On an actual change, adopt the value **and** call `router.refresh()` — the unifier that re-runs the server tree so the layout count and the list (if the user is on it) update live too.
   - Keep a **prop-sync effect** (`useEffect(() => adopt(requestCount), [requestCount])`) so the existing `router.refresh()`-driven decrement (approve/decline) is preserved rather than regressed — the server-recomputed prop is adopted as authoritative when it changes.
   - Guard the `router.refresh()` behind an actual-change check (compare against a ref) so poll → refresh → re-run-layout → prop-sync converges instead of looping.

Rule of thumb: **`force-dynamic` fixes server-component *pages*; it never makes a *layout* re-run on soft nav. Frozen layout-sourced values need a client poll (+ `router.refresh()`), not a route-segment flag.**

## What NOT to waste time on

- Adding `export const dynamic = "force-dynamic"` (or `revalidate = 0`) to the **layout** to refresh a layout-sourced value on soft nav. It will not work; the layout still won't re-invoke on client-side sibling navigation. Confirmed this session before building the poll.
- Chasing the Router Cache / `staleTime` knobs for the badge. Those govern segment payloads, not whether the layout function re-runs.
- Hard-refresh testing. It hides the bug because a full load re-runs the layout. Always test by navigating within the app.
- Over-engineering the badge into a server push / websocket. For a low-stakes count, a poll + focus/visibility listeners is sufficient and far cheaper; the count is an eventual-consistency badge.

## When to apply again

- Any time a value displayed in the persistent chrome (nav badges, counts, header state) is computed in a `layout.tsx` and must reflect changes that happen without a navigation the user initiates (idle arrival, another tab, an external/unauthenticated write). Move the freshness to a client poll + `router.refresh()`; do not reach for a layout segment flag.
- General App Router heuristic: before "fixing" staleness with a `dynamic`/`revalidate` flag, identify whether the stale value is sourced in a **page** (flag works) or a **layout** (flag does not work on soft nav — needs client-side refresh).