# PEL-004 — Turbopack JSX `Unexpected token` errors mislead about actual fault location

**Date:** May 13, 2026
**Project:** HarmonyDesk Pro (Cat 2B, Vercel deploy unblock)
**Stack:** Next.js + Turbopack on Vercel
**Severity:** Build-blocking — every commit after the regression fails to deploy until fixed

---

## Symptom

Vercel build fails with:

```
> Build error occurred
Error: Turbopack build failed with 1 errors:
./src/app/(dashboard)/<route>/page.tsx:LINE:COL
Unexpected token. Did you mean `{'>'}` or `&gt;`?
```

The error points at a specific `>` character at a specific column. The flagged character is **not** the actual bug — the bug is elsewhere in the file, usually upstream of the flagged position.

A correlated downstream symptom on Vercel: once a build fails, subsequent commits that don't touch the broken file also fail (because they inherit the broken state). Production stays pinned to the last successful build, and a hard refresh on production shows no change — there is no newer build to refresh to.

---

## Root cause (general)

Turbopack's JSX parser flags the first `>` it encounters in what it believes is **JSX text content**. The parser arrived at that conclusion because something further up the file caused it to lose JSX-opening-tag context. The flagged `>` is the downstream consequence, not the source of the bug.

The error message — "Did you mean `{'>'}` or `&gt;`?" — is the giveaway. Turbopack only suggests escaping when it thinks you're trying to write a literal `>` inside JSX children, which is only possible when the parser is no longer inside an opening tag's attribute list.

---

## Sub-pattern PEL-004a — Lone `>` on its own line

A multi-attribute JSX opening tag has its closing `>` placed alone on its own line:

```jsx
<a
  href={url}
  target="_blank"
  rel="noopener noreferrer"
  className="..."
  >
  {text}
</a>
```

Turbopack rejects the standalone `>`. The error points at line N column 21 (or wherever the lone `>` sits).

**Fix:** keep `>` adjacent to the last attribute on the same line:

```jsx
className="...">
```

---

## Sub-pattern PEL-004b — Missing opening tag (orphan attribute block)

The opening tag is missing entirely. Attributes float between a parent's opening and a child's closing:

```jsx
<dd className="...">

  href={session.meetingUrl}
  target="_blank"
  rel="noopener noreferrer"
  className="...">
  {session.meetingUrl}
</a>
</dd>
```

Turbopack sees `<dd>` open, then attempts to parse `href={...}`, `target="..."`, etc. as JSX text content (`{...}` expressions are valid in text). When it hits the `>` at the end of the `className` line, it has no opening tag to close — so it flags that `>` as an unexpected token.

The closing `</a>` further down is a tag closing something that was never opened — but Turbopack errors before reaching it.

**Fix:** add the missing opening tag (in this case `<a`) on the blank line above the attribute block.

---

## Diagnostic technique

When this error appears, **do not** trust the line/column as the actual fault location. Instead:

1. Open the file at the flagged line.
2. Read 10–20 lines upstream of the flagged position.
3. Look for one of:
   - A lone `>` on its own line (sub-pattern A)
   - A missing opening tag where attributes appear orphaned (sub-pattern B)
   - A stray `<` or `>` elsewhere in the block
   - A prematurely self-closed tag (`<tag />` where `</tag>` follows)
4. Identify the JSX-opening-tag-context loss, fix the structural problem.

If your fix moves the error to a new line/column but the same error message, the original structural problem is still present — you've only shifted Turbopack's first-encountered bad-`>` position.

---

## Canonical correct form for multi-attribute JSX

```jsx
<a
  href={url}
  target="_blank"
  rel="noopener noreferrer"
  className="...">
  {text}
</a>
```

- Opening tag `<a` on its own line, no attributes after it
- Attributes one per line, indented one level deeper than the tag name
- Closing `>` adjacent to the **last** attribute on the same line — never on its own line
- Children on subsequent lines
- Closing `</a>` aligned with the opening `<a`

---

## Operational consequences (Vercel)

When a JSX file in main breaks the Turbopack build:

- Production stays pinned to the last successful build's commit SHA — visible in Vercel **Deployments** tab as the row marked **Current**.
- Subsequent commits that don't touch the broken file still fail to build, because Turbopack rebuilds the whole project.
- Hard refreshing production does nothing — production is correctly serving the latest successful build.
- The fix is always a new commit to main that resolves the structural JSX issue. A "Redeploy" of the broken commit will re-run the same broken build and fail again — redeploys do not pick up new commits, only re-run an existing one.

If a fix commit is pushed but Vercel does not show a new deployment row within ~60 seconds:

- Check the GitHub commits view for a CI status icon on the new commit. No icon at all (not even a red X) means the webhook never fired.
- Quickest unblock: push a trivial no-op commit (e.g., a single space added to `README.md`) to force a fresh webhook delivery. The new build will pick up both commits.
- If even the trivial commit doesn't trigger a deployment: Vercel → **Settings** → **Git** to verify integration health.

---

## Related incidents

| Project | Date | Commit | Sub-pattern |
|---|---|---|---|
| HarmonyDesk Pro | May 12, 2026 | `2be8be4` ("Refactor session types and update status handling") | PEL-004a |
| HarmonyDesk Pro | May 12, 2026 | `76a1b57` ("Fix formatting of meeting URL link in calendar page") — pre-state, after partial fix | PEL-004b |
| HarmonyDesk Pro | May 12, 2026 | `997021b` ("Fix formatting of meeting URL link in calendar page") — partial fix; full resolution required a subsequent commit adding the missing `<a` opening tag | PEL-004b |

---

## Prevention

- When writing or editing multi-attribute JSX by hand, keep the closing `>` adjacent to the last attribute on the same line.
- When deleting or refactoring JSX, verify the matching opening tag still exists before committing — particularly when removing or restructuring nested elements.
- If editing in the GitHub web UI without a JSX-aware linter, scroll up and confirm the surrounding tag structure is intact before committing.
