# PEL-015 — PM2 Environment Refresh Requires Re-Reading the Ecosystem File, Not the --update-env Flag

**Date logged:** June 3, 2026
**Project:** HarmonyDesk Pro (EC2 backend, PM2 process management)
**Related entries:** This PEL retires Cat 3D Lesson 2 ("plain pm2 restart doesn't pick up .env changes; teardown cycle required") and replaces it with the canonical operation `pm2 reload <ecosystem-file>`. Forward references: Cat 3F handoff doc for the empirical verification trail and surrounding doctrine updates.

---

## Symptom

During Cat 3F's empirical baseline test of Cat 3D Lesson 2, the following sequence was run on the harmonydesk EC2 box (`ubuntu@18.221.48.232`):

1. `.env` modified — `DEMO_INBOX=cat-3f-baseline-probe-001` (replacing previous value `duelgregory@gmail.com`).
2. `pm2 restart harmonydesk` (plain restart) — succeeded; app online, fresh PID, restart count incremented.
3. `pm2 env 0 | grep DEMO_INBOX` — returned `DEMO_INBOX: duelgregory@gmail.com` (the OLD value).

This confirmed Cat 3D Lesson 2 in symptom. Hidden inside the PM2 restart output was a hint:

```
Use --update-env to update environment variables
[PM2] Applying action restartProcessId on app [harmonydesk](ids: [ 0 ])
```

The hint was tested next: `pm2 restart harmonydesk --update-env`. Result: `pm2 env 0 | grep DEMO_INBOX` still returned `duelgregory@gmail.com`. The flag PM2 had hinted at did not solve the problem.

A third variant was tested: `pm2 reload /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs` (passing the file path rather than the app name, with no flag). Result: `pm2 env 0 | grep DEMO_INBOX` returned `cat-3f-baseline-probe-001`. The reload picked up the new value cleanly. A second cycle (restoring `.env` to the original value and reloading again) confirmed the behavior in both directions: file-path-targeted reload picks up `.env` changes; plain restart and `--update-env` do not.

## Root cause

PM2 caches environment state at two independent layers, both of which must be invalidated for `.env` changes to propagate to the spawned process.

**Layer 1 — PM2 daemon's own `process.env`.** Populated when the PM2 daemon process is started. The `ecosystem.config.cjs` file is evaluated by PM2 when `pm2 start ecosystem.config.cjs` runs, and the file's module-level code (in HarmonyDesk Pro's case, `dotenv.config({ path: path.join(__dirname, ".env") })` at line 7) loads `.env` into the evaluator's `process.env`. From that moment forward, PM2's daemon env is frozen at the values `.env` held at original `pm2 start` time. Changes to `.env` on disk do not propagate into the daemon's env unless the daemon re-reads the file.

**Layer 2 — per-app env block evaluation.** The `env: { ... }` block in `apps[0]` (lines 16-39 in HarmonyDesk Pro's ecosystem.config.cjs) is JavaScript that references `process.env.X || "fallback"` and produces concrete values. PM2 evaluates this block once at `pm2 start` time and stores the resulting concrete env object in its in-memory app definition. Plain `pm2 restart` re-spawns the process using this stored concrete env; it does not re-evaluate the block, so changes to either `.env` OR the ecosystem file would not propagate.

**Why each tested command did or did not refresh:**

- `pm2 restart <name>` — re-spawns the process. Does not re-evaluate the env block (still cached from `pm2 start`). Does not re-read PM2 daemon env. Both cache layers preserved. Result: stale env.

- `pm2 restart <name> --update-env` — re-spawns the process AND re-applies PM2 daemon's current `process.env` to the spawn env. But PM2 daemon's env is itself stale (Layer 1 cached). So `--update-env` re-applies stale values. The flag name implies "update from environment files" but the actual behavior is "re-apply cached daemon env." Result: stale env.

- `pm2 reload <ecosystem-file>` — re-reads the ecosystem file from disk. That re-evaluation runs the module-level `dotenv.config()` (refreshing PM2 daemon's `process.env` from current `.env`, invalidating Layer 1) AND re-evaluates the env block (producing fresh concrete values, invalidating Layer 2). Both cache layers invalidated. The spawned process gets the new env. Result: fresh env.

The teardown cycle (`pm2 delete && pm2 start <file> && pm2 save`) works for the same reason `reload <file>` works — `pm2 start <file>` re-reads the file. It's just heavier than `reload <file>`, with no functional benefit for the env-refresh use case.

## Diagnostic technique

When a PM2-managed process is not picking up `.env` changes:

1. **Identify which cache layer matters for the variable in question.**
   - If the variable is referenced in the ecosystem env block (e.g., `RESEND_API_KEY`, `EMAIL_FROM`, `DEMO_INBOX` in HarmonyDesk Pro): PM2 sets it in the spawn env. Cached at `pm2 start` time. Refresh requires re-reading the file.
   - If the variable is NOT in the ecosystem env block (e.g., `STRIPE_SECRET_KEY`, `SUPABASE_SERVICE_ROLE_KEY`): PM2 doesn't set it. The app's runtime `dotenv.config()` (in `server.js` line 30 for HarmonyDesk Pro) loads it at every spawn. Plain `pm2 restart` already picks up changes for these vars at the application layer, though `pm2 env 0` won't reflect them (because pm2 env reports spawn env, not the app's runtime `process.env`).

2. **For block-referenced vars, use `pm2 reload <ecosystem-file>`** with an absolute file path. Example for HarmonyDesk Pro:
   ```
   pm2 reload /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs
   ```
   Verify the refresh worked via `pm2 env 0 | grep VAR_NAME`. Expect to see the new value.

3. **Do not use `pm2 restart --update-env` as a substitute.** The flag re-applies cached daemon env; it does not re-read the file. If the daemon env is stale, `--update-env` is a no-op for the user's actual need.

4. **Do not use the teardown cycle unless reload fails.** Teardown works but is heavier and has more downtime. Reload achieves the same refresh at roughly the same restart cost (~2-3 seconds in fork mode).

## Canonical fix

**For all future env refreshes on HarmonyDesk Pro (and any DuelTech project using a similar PM2 + ecosystem.config.cjs + module-level dotenv pattern), the canonical operation is:**

```
pm2 reload /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs
```

This replaces the prior canonical operation (`pm2 delete harmonydesk && pm2 start /apps/harmonydesk/harmonydesk-mvp/ecosystem.config.cjs && pm2 save`). Properties of the new operation:

- Re-reads `ecosystem.config.cjs` from disk fresh on every invocation.
- Re-evaluates the module-level `dotenv.config()` — loads current `.env` into the evaluator's `process.env`, invalidating Layer 1.
- Re-evaluates the `apps[0].env` block — produces fresh concrete env values, invalidating Layer 2.
- Spawns the app with the new env.
- Does not require a separate `pm2 save` (the process list itself is unchanged — same app name, same configuration, only env contents differ).
- Roughly the same downtime as `pm2 restart` (~2-3 seconds in fork mode for HarmonyDesk Pro).
- Same risk profile as restart: if `ecosystem.config.cjs` or `.env` is malformed, the reload fails and the app may not come back up. Save backups before any non-trivial edit, same as for restart.

The teardown cycle remains valid as a fallback if `reload <file>` ever fails, but `reload <file>` is the default.

## Operational consequence

For the Cat 3F arc: the proposed structural change to `ecosystem.config.cjs` (strip the line-7 dotenv call) is no longer needed. Cat 3F-2 collapses from a production-risk code change (full-file replacement, deploy, teardown to apply, risk of the app failing to come back up if the env contract was subtly different than assumed) to a one-line operational doctrine update. Total time saved on Cat 3F closeout: ~30 minutes of structural surgery plus the deferred risk of touching the ecosystem file in production.

For the rest of the secret-rotation queue (`STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY`, `SUPABASE_SERVICE_ROLE_KEY`): each rotation that touches a block-referenced var (notably `RESEND_API_KEY`, the only block-referenced secret in the queue) now uses `pm2 reload <file>` instead of teardown. The savings per rotation are marginal in absolute terms (~5 seconds of additional downtime in teardown vs. reload), but the removal of operational ceremony — no more "delete, start, save" three-step — and the elimination of the `pm2 save` step are meaningful for clarity.

For the Lessons Ledger more broadly: Cat 3D Lesson 2 was correct in observation but incomplete in framing. Lessons that describe symptoms ("plain restart doesn't work") without mechanism ("because PM2 caches at two layers and `pm2 restart` invalidates neither") can imply heavier remediations than are necessary. PEL-015 argues for mechanism-depth on operational lessons going forward — describe what the cache is, what invalidates it, and why each tested command did or did not invalidate it. Without the mechanism, future arcs inherit the heavier workaround and never re-test whether it's still the right answer.

## Recurrence prevention

- **Doctrine update**: any documentation that references "PM2 env refresh = teardown cycle" should be updated to "PM2 env refresh = `pm2 reload <ecosystem-file>` (absolute path)." Cat 3D Lesson 2 is retired in favor of this PEL; future SSPs and handoffs should reference PEL-015 rather than Cat 3D Lesson 2.

- **SSP discipline for env-rotation arcs**: the env-refresh step in any future env-touching SSP is `pm2 reload <file>`, not `pm2 delete && pm2 start && pm2 save`. Teardown remains valid as a fallback but isn't the default.

- **Future-PEL prompt for ambiguous CLI hints**: when PM2 (or any tool) emits a hint like "Use --update-env to update environment variables," do not assume the hint is correct for your configuration. The hint may describe a different problem than the one you have. Test the hint, and also test the structurally adjacent commands (`reload <name>`, `reload <file>`, `reload <name> --update-env`). The "right" command for your refresh is the one that empirically refreshes, not the one the tool hinted at.

- **Verification methodology for env-touching work**: include an empirical verification cycle (modify a non-critical env var, refresh, observe `pm2 env 0`, restore) BEFORE rotating actual secrets. This is the methodology Cat 3F used; it should be standard for any env-touching arc going forward. The cost is one extra `.env` edit and ~30 seconds of restart time; the benefit is empirical confirmation that the chosen refresh command actually refreshes the right thing in the current configuration.

## Related context

This PEL was surfaced during Cat 3F's empirical baseline test, which was itself prompted by the realization that Cat 3D Lesson 2 had been carried forward into Cat 3F's SSP as the canonical workaround needing retirement. The original Cat 3F-2 phase was designed to retire Lesson 2 via structural change to `ecosystem.config.cjs` — a multi-step production-risk operation. The actual retirement happened via empirical exploration of PM2's CLI surface: `pm2 restart`, `pm2 restart --update-env`, `pm2 reload <ecosystem-file>`. The last variant worked; the structural change was never needed.

The PEL-level lesson beyond the specific PM2 mechanics: when a documented workaround has been carried across multiple lessons or sessions without anyone re-testing the underlying mechanism, the workaround may be either heavier than necessary or addressing a phenomenon that doesn't exist in the form described. A single empirical test cycle (~5 minutes in this case) can collapse weeks of derived process into the actual minimum operation. The cost of running such a test is small; the cost of not running it compounds across every downstream arc that inherits the workaround.

A secondary lesson: PM2's own CLI hint was actively misleading in this configuration. "Use --update-env to update environment variables" is true in some PM2 configurations (where daemon env is the relevant cache layer) but false in HarmonyDesk Pro's configuration (where the ecosystem file is the relevant cache layer). Tool hints are written for a typical case; the typical case may not be your case. Treat hints as evidence, not instructions.
