# Lesson — Mode C Ops Gateway Bootstrap

## Date

2026-06-18

## Project

DuelTech Opportunity Engine

## Summary

Mode C Ops Gateway was built and verified for the DuelTech Opportunity Engine.

The new operating pattern is:

```text
ChatGPT
→ GitHub request artifact
→ clean control node runner
→ Opportunity Engine EC2
→ GitHub result artifact
→ ChatGPT reads result
```

This replaces the slower manual relay loop where Duel copied commands and EC2 output between AI agents and infrastructure.

## What made it work

The successful design used GitHub as the shared control plane because ChatGPT already had GitHub access.

The control node polls GitHub, validates request files against an allowlist, executes bounded actions, writes Markdown results, and pushes those results back to GitHub.

This avoids unrestricted shell access while still allowing useful EC2 operation.

## Key implementation choices

- Use a clean control node, separate from laptop files and other agent workspaces.
- Use GitHub request/result artifacts instead of chat-pasted command output.
- Use a strict action allowlist.
- Keep persistent PM2 worker mode as `connected-dry-run`.
- Allow one-off live previews only through explicit HTTPS URL allowlists.
- Write result artifacts back to GitHub for auditability.

## Verified actions

```text
repo_status: PASS
build_check: PASS
runner installed: yes
systemd timer active: yes
result artifacts written to GitHub: yes
```

## Important distinction

Mode C is not the product objective.

Mode C is the execution rail.

The product objective remains building the Opportunity Engine, especially source-to-signal logic and lead-discovery calibration.

## Lesson

Agent infrastructure is useful only when it reduces relay friction without increasing blast radius.

The right pattern was not "give the AI shell access."

The right pattern was:

```text
bounded action → auditable request → controlled execution → durable result artifact
```

## Reuse guidance

This pattern can be reused for other portfolio engines when the required action set is narrow and can be allowlisted.

Do not reuse it as a general-purpose command bridge.

Use it only when:

```text
allowed actions are explicit
results are durable
secrets are not exposed
agent scope is bounded
humans can audit the artifact trail
```

## Follow-up

The next DuelTech Opportunity Engine work should return to calibration:

```text
Improve source-to-signal logic.
Sharpen accessible buyer evidence recognition.
Sharpen direct operator pain extraction.
Keep blocked / low-quality sources visible for audit but excluded from pre-candidate support.
```
