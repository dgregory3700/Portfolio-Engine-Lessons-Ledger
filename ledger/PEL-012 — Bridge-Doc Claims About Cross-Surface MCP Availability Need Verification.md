# PEL-012 — Bridge-Doc Claims About Cross-Surface MCP Availability Need Verification, Not Recitation

**Date logged:** June 3, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop, claude.ai web, MCP infrastructure)
**Related entries:** PEL-011 (Claude's Diagnostic Blind Spots in MCP Environments). PEL-012 is a direct recurrence of PEL-011's instance #3, this time propagated through a handoff doc rather than fired within a single session.

---

## Symptom

The session bridge from Cat 3E → Cat 3F (written in Claude Desktop during the SSH MCP setup arc on 2026-06-02) included this assertion:

> GitHub MCP from claude.ai web (newly recognized, not new). GitHub MCP was always accessible via tool_search from claude.ai web; the SSH MCP arc was where this got surfaced. Direct commits, file reads, branch operations from any Claude surface, not just Claude Desktop.

When the Cat 3F session opened in claude.ai web on 2026-06-03, the receiving Claude:

1. Inspected its visible tool list and did not see GitHub MCP tools.
2. Reported to the user: "My available tool set in this session does not include SSH MCP or GitHub MCP."
3. Did NOT call `tool_search` for "github" before making that report.

The user, prompted by both the bridge doc's claim and the receiving Claude's contradiction, switched to Claude Desktop. In Desktop, both MCPs were available (after the Docker reattach issue documented in PEL-013), and Cat 3F proceeded. Whether the bridge's claim about web-surface accessibility was actually correct was never empirically tested by either Claude.

## Root cause

Two distinct failures stacked:

**Bridge-doc author failure.** The bridge stated a capability claim ("GitHub MCP accessible via tool_search from claude.ai web") that was not directly verified in the authoring session. The phrasing "the SSH MCP arc was where this got surfaced" implies that empirical verification happened in some prior session, but no evidence (tool_search query, response) was carried forward. The claim entered the bridge as belief, not as observation.

**Receiving-session failure.** The receiving Claude in claude.ai web read the bridge, noted the claim, and then — when its own visible tool list didn't contain GitHub tools — reported the tools as absent without running the very `tool_search` call the bridge had claimed was the access mechanism. This is the exact pattern PEL-011 instance #3 documented within a single session; PEL-012 is that pattern crossing a session boundary.

The compounding effect is the meaningful one. PEL-011's three instances were all single-session reasoning failures. Once a handoff doc encodes an unverified capability claim, every downstream session inherits it. If those sessions fail to probe, the claim propagates indefinitely without ever being tested. Multi-session arcs accumulate these unverified claims at a small but nonzero rate per bridge written, and the cost of acting on a wrong one is non-trivial (in this arc: a surface switch and ~10-15 minutes of arc time before MCPs were operational).

## Diagnostic technique

For users debugging multi-session AI workflows: treat capability claims in bridge docs (X is accessible from surface Y) as hypotheses, not facts. A single `tool_search` call from the receiving session usually settles whether the claim holds. If the receiving Claude reports a tool as missing, ask whether `tool_search` was called before that report was made.

For Claude sessions receiving a bridge doc: capability claims in the bridge are inputs to verification, not conclusions. Before reporting any tool as missing or any capability as unavailable, run `tool_search` with relevant keywords. If the search returns nothing, the absence is confirmed; if the search returns tools, the bridge was right (or the visible-tools heuristic was incomplete, which is the structural condition PEL-011 documents).

For bridge-doc authors: distinguish empirically tested claims from inferred claims. A claim like "GitHub MCP is accessible from claude.ai web via tool_search" should be backed by an actual `tool_search` query run in the authoring session, with the result pasted as evidence. If the claim is inferred from documentation or another session's report, mark it as "ASSUMED, NOT TESTED" so downstream readers know to verify before relying.

## Canonical fix

Procedural, at the doc-template and session-open levels:

- **Bridge / SSP template addition**: a "Verified capabilities" subsection that requires the author to either paste evidence of testing (tool call + response) or explicitly mark a claim as untested. Claims without evidence default to untested.

- **Session-open gates addition**: for any capability claim the bridge makes, run a probe in the receiving session before relying on it. This was implicit in Cat 3F's SSP ("Re-run the end-of-last-session tests"); making it explicit catches the case where a bridge claims a capability that prior sessions never actually exercised.

- **Standing instruction for any Claude session interacting with MCP tools**: before answering "I don't have X tool" or "I can't do Y," call `tool_search` with relevant keywords. Treat `tool_search` as the first move, not the last. This is the same instruction from PEL-011's recurrence prevention; PEL-012 demonstrates why the instruction needs to fire at session-open even when the bridge doc has done some of the homework.

## Operational consequence

For this arc: ~2 turns of conversation were spent debating "path A: manual fallback vs path B: wire MCPs first" when the actual answer (probe with tool_search, then decide) would have collapsed both paths to one. Worse, the receiving Claude's "tools missing" report — issued without tool_search — primed the user to switch surfaces. The switch turned out to be the right outcome for unrelated reasons (Docker-backed GitHub MCP needed reattach anyway, documented in PEL-013), but the decision to switch was driven by an unverified claim, not by evidence.

The broader cost: every bridge doc accumulates capability claims at some rate, and unverified claims will be wrong with some probability. The verification cost (one tool_search call) is essentially zero; the cost of routing around a wrong claim compounds across the arc. PEL-012 argues for paying the verification cost reflexively at session-open, every time, for every claim the bridge makes about tooling.

## Recurrence prevention

- SSP / bridge template update: add a "Verified capabilities" subsection. Claims with paired evidence; claims without get explicit "ASSUMED, NOT TESTED" markers.
- Session-open gate addition: "Probe each capability claim the bridge makes via the corresponding tool call before relying on it. tool_search before reporting any tool as missing."
- For receiving sessions: if a bridge claim contradicts the receiving session's visible tool list, run tool_search to resolve the contradiction empirically, not by picking which source to believe. The probe is the tiebreaker.

## Related context

This PEL is the cross-session counterpart to PEL-011 instance #3. PEL-011 documented the pattern as a single-session reasoning failure ("this Claude failed to call tool_search before claiming GitHub MCP was missing"). PEL-012 documents the same pattern as a multi-session propagation failure ("the bridge encoded an unverified claim; the receiver inherited it and also didn't probe"). The lesson is structurally the same; the operational cost compounds across sessions in a way the original PEL-011 didn't fully capture.

The strongest data point in this entry, mirroring PEL-011's own observation about itself: the bridge that encoded the unverified claim was written in a session that had just shipped PEL-011. The session that received the bridge was a Claude that had been instructed (via system prompt and operational doctrine) to probe before reporting tools missing. The pattern fired anyway. This argues, as PEL-011 did, that the remedy lives in workflow scaffolding (explicit probes at session-open, structured "Verified capabilities" sections in bridges) rather than relying on individual model runs to remember the lesson.
