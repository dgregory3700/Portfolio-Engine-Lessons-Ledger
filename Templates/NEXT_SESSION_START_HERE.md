# Next Session Start Here

> This file is the authoritative entry point for the next Chessa session.
>
> Update or rewrite it only during final session closeout, after the current work, validation, record reconciliation, and next-task decision are complete.
>
> Do not rewrite this file in the middle of an active session merely because an intermediate task, pull request, merge, or validation step completes.

## Read in this order

1. `STATUS_GOVERNANCE.md`
2. `build-status/CURRENT_STATUS.md`
3. `build-status/NEXT_TASK.md`
4. `build-status/STAGE_3_DISCOVERY_EFFECTIVENESS_AND_HUNTING_GROUND_AUDIT.md`
5. `[CURRENT TASK-SPECIFIC PLAN OR CONTROL RECORD]`
6. `ssps/[LATEST-DATED-SESSION-START-PROTOCOL].md`
7. `handoffs/[LATEST-DATED-HANDOFF].md`

Confirm all seven sources were read before proposing, authorizing, or performing work.

Treat these sources as the current repository checkpoint, subject to the authority order in `STATUS_GOVERNANCE.md`.

Historical SSPs, old handoffs, superseded audits, generated artifacts, stale branches, draft pull requests, and prior completion claims do not override current runtime evidence or current committed repository state.

## Current session objective

**Primary objective:**
[STATE ONE BOUNDED OBJECTIVE FOR THE NEXT SESSION.]

This objective must directly support Chessa’s path toward reliable real-world operation.

Do not substitute documentation activity, planning, calibration, or repository motion for the actual objective unless one of those is the explicitly bounded task.

## Current checkpoint

**Date reconciled:** `[YYYY-MM-DD]`

**Repository state:**

* GitHub `main`: `[FULL COMMIT SHA]`
* Relevant merged pull request(s): `[PR NUMBERS AND MERGE COMMITS]`
* Repository validation: `[PASSED / FAILED / NOT RUN / NOT APPLICABLE]`
* Relevant GitHub Actions evidence: `[RUN AND JOB REFERENCES]`

**Runtime state:**

* EC2 checkout: `[PATH, BRANCH, AND COMMIT]`
* GitHub-to-EC2 parity: `[VERIFIED / ABSENT / PARTIAL / UNVERIFIED]`
* PM2 state: `[EXACT VERIFIED STATE]`
* Worker state: `[RUNNING / PAUSED / ABSENT / UNVERIFIED]`
* Latest runtime evidence: `[DATED CAPTURE OR ARTIFACT]`

**Audit state:**

* Stage 1: `[STATUS]`
* Stage 2: `[STATUS]`
* Stage 3: `[STATUS]`

**Remediation state:**

* Priority 0: `[STATUS]`
* Priority 1: `[STATUS]`
* Priority 2: `[STATUS]`
* Priority 3: `[STATUS]`
* Priority 4: `[STATUS]`
* Current gate or deployment phase: `[STATUS]`

Clearly distinguish:

* merged in GitHub
* repository-validated
* deployed or synchronized to EC2
* runtime-verified
* supported by fresh real-run evidence

A GitHub merge does not prove deployment. Repository validation does not prove runtime correctness. Runtime execution does not by itself prove discovery effectiveness.

## What was completed in the prior session

* `[COMPLETED ITEM]`
* `[COMPLETED ITEM]`
* `[COMPLETED ITEM]`

For each material completion, include the authoritative evidence:

* pull request and merge commit
* validation command or GitHub Actions run
* EC2/runtime capture
* generated artifact
* audit conclusion
* operator decision

Do not report planned, attempted, drafted, or partially validated work as completed.

## What changed

* `[REPOSITORY CHANGE]`
* `[RUNTIME CHANGE, OR “NO RUNTIME CHANGE”]`
* `[GOVERNANCE OR DOCUMENTATION CHANGE]`
* `[AUDIT OR AUTHORIZATION CHANGE]`

State explicitly when a category did not change.

## What remains unresolved

1. `[UNRESOLVED ITEM]`
2. `[UNRESOLVED ITEM]`
3. `[UNRESOLVED ITEM]`

Do not reopen resolved questions without new concrete evidence.

## Single next bounded task

**Task:**
[STATE ONE SPECIFIC, EXECUTABLE TASK.]

**Why this is next:**
[EXPLAIN HOW IT FOLLOWS FROM THE VERIFIED CHECKPOINT AND MOVES CHESSA TOWARD PRODUCTION READINESS.]

**Required evidence:**

* `[EVIDENCE INPUT]`
* `[EVIDENCE INPUT]`
* `[EVIDENCE INPUT]`

**Expected output:**

* `[CODE CHANGE, REPORT, MATRIX, RUNTIME CAPTURE, VALIDATION RECORD, OR DECISION]`

This section must identify actual work. Do not use vague instructions such as:

* continue Gate A
* proceed with deployment
* inspect the repository
* review status
* get ready
* determine next steps

## Authorization boundaries

### Authorized

* `[SAFE INSPECTION OR REPOSITORY WORK]`
* `[REVERSIBLE VALIDATION]`
* `[DOCUMENTATION REQUIRED BY THE TASK]`

### Not authorized without Duel’s separate approval

* spending money
* paid provider calls outside an explicitly approved bounded run
* external outreach
* public publishing
* revealing or changing secrets
* destructive deletion
* irreversible production changes
* material database or Supabase mutation
* candidate or Admission mutation
* Portfolio Engine Ledger promotion
* disabling safeguards
* EC2 synchronization or deployment
* dependency installation or build on EC2
* PM2 start, restart, reload, stop, resurrect, or configuration
* controlled real live research
* Stage 3 re-audit

Remove an item from this prohibited list only when the next session has explicit authorization for that exact action.

## Do not do

* Do not reintroduce dry-run, mock-run, or connected-dry-run paths.
* Do not treat a merge as deployed proof.
* Do not treat generated artifacts as current runtime truth without verification.
* Do not change the Stage 3 verdict without completing the re-audit gate.
* Do not reopen completed priorities or prior architecture decisions without new evidence.
* Do not modify `NEXT_SESSION_START_HERE.md` during ordinary mid-session progress.
* Do not create a new SSP or handoff merely to restate unchanged information.
* Do not leave stale current-state language in any governing or operator-facing record.

## Definition of done

The next bounded task is complete only when:

1. `[IMPLEMENTATION OR INVESTIGATION REQUIREMENT]`
2. `[VALIDATION REQUIREMENT]`
3. `[EVIDENCE REQUIREMENT]`
4. `[RECORDING REQUIREMENT]`
5. `[AUTHORIZATION-BOUNDARY REQUIREMENT]`

A task is not complete merely because a pull request was opened, merged, or locally validated.

## Re-audit gate

Stage 3 remains `[FAIL / RE-AUDIT REQUIRED]` until all required conditions are established by evidence.

Re-audit only after:

1. All material Stage 3 repairs required for the re-audit are merged and repository-validated.
2. The intended GitHub version is separately authorized, safely synchronized to EC2, and parity-verified.
3. Deployment and rollback evidence are recorded.
4. The required runtime posture is verified.
5. A separately authorized controlled real live run produces fresh Priority 3 proof artifacts and Priority 4 discovery telemetry.
6. A fresh evidence-based audit evaluates discovery effectiveness and hunting-ground effectiveness.

Planning, repository preparation, documentation completion, or deployment alone does not change the Stage 3 verdict.

## Mandatory closeout

Before the next session ends, update the three living records named in `STATUS_GOVERNANCE.md`:

1. `build-status/CURRENT_STATUS.md`
2. `build-status/NEXT_TASK.md`
3. `NEXT_SESSION_START_HERE.md`

Create or update a dated SSP and handoff when the session leaves material work for another session.

`NEXT_SESSION_START_HERE.md` must be rewritten only after the next task and checkpoint are final for that session.

### Governance reconciliation

Inspect `STATUS_GOVERNANCE.md` before closeout.

Update it when:

* its current required facts are stale
* its next-task statement is stale
* the closeout protocol has changed
* the authority order or operating rules have changed
* the list of living records has changed
* a new required consistency rule has been adopted

When no change is required, record that it was reviewed and remains current in the closeout evidence.

### Mandatory conditional record check

Before closeout, inspect each of these files and either update it or explicitly record why no update is required:

* `README.md`
* `USER_MANUAL.md`
* `INDEX.md`
* `STATUS_GOVERNANCE.md`
* `build-status/STAGE_3_DISCOVERY_EFFECTIVENESS_AND_HUNTING_GROUND_AUDIT.md`
* the current task-specific plan or control record
* the latest dated SSP
* the latest dated handoff

Apply these rules:

| Session change                                                                           | Records that must be reconciled                                                                                                                           |
| ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Runtime status, PM2 state, operator commands, deployment posture, or operating procedure | `USER_MANUAL.md`, `README.md`, `CURRENT_STATUS.md`, `NEXT_TASK.md`, `NEXT_SESSION_START_HERE.md`                                                          |
| Audit verdict, audit evidence, or remediation status                                     | Stage 3 audit record, `CURRENT_STATUS.md`, `NEXT_TASK.md`, `NEXT_SESSION_START_HERE.md`, and `INDEX.md` when its navigation or status summary is affected |
| Governance, closeout, authority, or source-of-truth rules                                | `STATUS_GOVERNANCE.md`, `NEXT_SESSION_START_HERE.md`, and validators or indexes that encode those rules                                                   |
| Architecture or technical contract                                                       | Relevant architecture/control record, `README.md`, and `INDEX.md`                                                                                         |
| Material work continues into another session                                             | A new dated SSP and dated handoff, referenced by `NEXT_SESSION_START_HERE.md`                                                                             |
| A current record is retired, renamed, or removed                                         | Remove or replace every live reference and update validators in the same change                                                                           |
| No material change                                                                       | Do not invent a new historical document; record the review result in the living closeout evidence                                                         |

### Final consistency check

Before calling the session complete:

1. Confirm the three living records agree.
2. Confirm `STATUS_GOVERNANCE.md` does not name an obsolete next task.
3. Confirm the latest SSP and handoff are present in the seven-item reading order.
4. Confirm `README.md`, `USER_MANUAL.md`, and `INDEX.md` were inspected.
5. Confirm GitHub state and EC2 runtime state are not conflated.
6. Confirm the audit verdict matches the audit record.
7. Confirm the next task is singular, specific, and authorized.
8. Run the repository’s applicable validation suite.
9. Inspect the applicable post-merge `main` validation before claiming repository closeout.
10. Only then finalize `NEXT_SESSION_START_HERE.md`.

## Session-start instruction

After reading the seven required sources, begin by stating:

1. the verified current checkpoint
2. the single next bounded task
3. the evidence that supports that task
4. the authorization boundary
5. what must not be reopened

Do not respond only with “ready.”
