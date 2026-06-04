# PEL-014 — Claude Desktop's MCP "Running" Status Badge Is Not Authoritative

**Date logged:** June 3, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop)
**Related entries:** PEL-006 (pm2 Silence Is Not Proof of Inactivity) — same family of lesson, different tool surface. PEL-013 (Docker-backed MCPs are wake-fragile) — the trigger condition that exposed this badge unreliability.

---

## Symptom

After a Claude Desktop restart prompted by a Docker-backed GitHub MCP reattach failure (PEL-013), the user opened Claude Desktop's developer settings panel to verify MCP state. Both MCP servers were listed:

- `github` — blue "running" badge
- `ssh` — blue "running" badge

The user reported the badges to Claude in chat. Claude probed both:

- `github:get_me` → "Tool not found."
- `ssh:exec` (whoami) → returned correctly.

The badge for github said "running" while the MCP was, in fact, not callable. The badge's "running" state was tracking some upstream signal (the docker container had been spawned, or the attach attempt had been initiated, or the server entry had been parsed from config) but not the property that matters for usage: "tools are attached and respond to calls."

## Root cause

The "running" badge in Claude Desktop's developer settings reflects an internal state machine inside Claude Desktop's MCP subsystem, not the end-to-end callable state of the tools the MCP server exposes. The badge can be in the "running" state when:

- The MCP server process has been spawned (Docker container started, Node binary launched) but the tool-registration handshake hasn't completed.
- The MCP server process is running but its connection to Claude Desktop's tool router is broken or stale (e.g., the stdio pipe was closed and re-opening hasn't completed).
- An attach attempt was initiated and hasn't yet been definitively marked failed.
- The MCP server's last successful attach occurred at some point during the lifetime of Claude Desktop, but a subsequent reconnect failure has not yet propagated to the UI badge.

In the Cat 3F arc, the most likely explanation is the second pattern: the Docker container for github-mcp-server was started (a `docker ps` check would have confirmed it), but the stdio pipe between Claude Desktop and the container wasn't actually carrying live tool calls. The badge tracked "container started" rather than "tools callable."

The general principle: any status display in a multi-process system has to choose what to track. The simplest signal (process exists) is fastest to compute and most reliably renderable; the strongest signal (tools respond) requires actively probing the server and waits for a round-trip response. UI status badges typically prioritize the former. This is reasonable engineering — the UI cannot afford to block on a slow probe every render — but it means the badge and the property the user cares about are decoupled.

## Diagnostic technique

For any claim about MCP state that originates from a UI element (Claude Desktop developer panel, Docker Desktop container list, OS process viewer), treat it as a hypothesis. The verifying check is:

1. For the suspected MCP, call its simplest tool (`github:get_me`, `ssh:exec` with `whoami`, etc.).
2. If the tool returns valid data, the MCP is genuinely callable. The badge was correct.
3. If the tool returns "Tool not found" or hangs, the MCP is not callable regardless of what the badge says. The badge was wrong.

This probe takes seconds. The UI badge can be wrong for indefinite periods, especially after wake/resume cycles where the badge state machine and the actual MCP state machine drift.

## Canonical fix

There is no Claude-Desktop-side fix because the badge semantics are an upstream design choice (and arguably the right one for UI render performance). The fix is procedural:

- **For users debugging MCP issues**: never report "the badge says X" as the definitive state. Pair it with a probe outcome from Claude. "The badge says running, and `github:get_me` returns my user object" is a strong signal. "The badge says running" alone is not.
- **For Claude sessions diagnosing MCP issues**: when a user reports a badge state, ask for a probe outcome before reasoning further. If the badge says one thing and the probe says another, the probe is the truth.

## Operational consequence

For the Cat 3F arc: when the user reported both badges as "running" after the Claude Desktop restart, Claude was momentarily incongruent — if both said running, why had the prior `github:get_me` returned "Tool not found"? The resolution was immediate (probe again, see what's actually callable, proceed) but a less careful diagnostic flow could have spent further turns assuming GitHub MCP was working and constructing unrelated explanations for tool-call failures.

The broader operational consequence is that any MCP-environment debugging conversation needs to factor in "UI says vs probe says" as a recurring asymmetry. This adds a constant small overhead to MCP diagnostic work that an unaware user/Claude pair would spend cycles on every time.

## Recurrence prevention

- **Standing instruction**: treat MCP UI status badges (Claude Desktop, Docker Desktop, OS process viewers) as one input among several. The probe is the deterministic check. When badge says running but probe says missing, the probe is correct.
- **Bridge-doc / SSP discipline**: never document MCP state as "running per the badge" without a paired probe outcome. Document as "running per the badge AND `get_me` returned valid user object" or "badge says running but probe returns not found — investigating." Document the discrepancy explicitly when it occurs; it's diagnostic signal.

## Related context

This is the MCP-domain analog of PEL-006's lesson about pm2 silence. Both share the structure: a UI status display was treated as authoritative when it actually tracked a weaker condition than the user cared about. In PEL-006, pm2's empty log output was taken as "service is fine" when it actually meant "no log writes have occurred recently" — which is consistent with both "service is fine" and "service is crashed and not writing logs." In PEL-014, the badge's "running" state is consistent with both "tools are callable" and "container started but tools dead."

The remedy in both cases is the same: substitute a probe of the actual property of interest for the convenient-but-weaker UI signal. This is a generalizable lesson about debugging in any system that uses status displays: identify what the display tracks vs. what you care about, and probe directly for the latter when they could diverge.
