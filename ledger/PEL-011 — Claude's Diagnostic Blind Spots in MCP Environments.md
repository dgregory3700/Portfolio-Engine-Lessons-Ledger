# PEL-011 — Claude's Diagnostic Blind Spots in MCP Environments

**Date logged:** June 2, 2026
**Project:** DuelTech AI tooling layer (Claude Desktop, Claude.ai web, MCP infrastructure)
**Related entries:** PEL-009 (Corporate SSL Inspection Silently Breaks Node MCP Attach) and PEL-010 (Native ssh.exe MCP Wrappers Fail Silently on Windows) cover external failure modes in the same SSH MCP setup arc. PEL-011 is the meta-PEL of that arc: the previous two cover external failure modes, this one covers Claude's reasoning *about* external failure modes — specifically the blind spots that fire predictably when MCP servers are involved.

---

## Symptom

Three distinct patterns observed in the same multi-hour SSH MCP setup arc, each producing a measurably wrong diagnosis or step:

**1. Misidentification of execution surface.** A Claude Desktop session, after the SSH MCP failed to attach (PEL-009), introspected its tool catalog, didn't find SSH tools, and concluded *"I must be on claude.ai web, not Claude Desktop — that's why I don't have SSH tools."* This was wrong. A "Could not attach to MCP server ssh" popup had fired (visible to the user only), confirming both that the session WAS in Claude Desktop and that an SSH MCP entry was present in config but had failed to attach. Claude pattern-matched on "missing SSH tools = wrong surface" instead of "missing SSH tools = attachment failed."

**2. Enumeration without evidence-weighting.** A different Claude Desktop session, after the SSH MCP attached but its calls failed (PEL-010), listed plausible AWS-side causes for the exit-255 failure: instance stopped, public IP changed without an Elastic IP, security group blocking the user's current IP. The user had — minutes before — confirmed Termius worked from the same laptop to the same host. That single fact ruled out all three candidates Claude listed. Claude generated the list from a generic "ssh exit 255 troubleshooting" template rather than weighing the conversation's existing evidence.

**3. Tool-surface introspection failure (self-instance).** This Claude (claude.ai web), after drafting PEL-009 and preparing to commit it, instructed the user to relay the markdown to Claude Desktop "because Claude Desktop has the GitHub MCP." The user pointed out that this Claude had GitHub access via its own deferred tool surface — which a single `tool_search("github")` call would have confirmed in seconds. The instruction to relay was constructed without ever making that call. The capability lived one tool query away; the assumption that it lived elsewhere was unverified.

Each instance felt locally reasonable to the Claude that produced it; each was visibly wrong against evidence either present in the conversation or one tool call away.

## Root cause

Claude's view of its own runtime environment is partial in three specific ways that conspire to produce confident-wrong answers when MCP servers are involved:

**1. No popup or out-of-band UI visibility.** Claude Desktop renders attachment-failure toasts as OS-level UI notifications. Browser-based clients render various messages outside the chat surface. Claude — running inside the conversation — has no input from any of these. When a popup is the only signal that something failed, Claude has no observation that says "an attachment failed"; it just sees a shorter-than-expected tool catalog and has to infer a cause.

**2. Deferred tool surfaces.** Many tools are loaded lazily via `tool_search` and don't appear in Claude's visible tool list until explicitly queried. The GitHub MCP in this arc was deferred — the entire 41-tool catalog was available, but invisible without a `tool_search` call. A Claude that introspects only the *visible* tool list and concludes "I don't have X" is reasoning from incomplete information. The procedural fix (always `tool_search` first) is in Claude's standing instructions, and Claude still skips it under certain conditions — particularly when there's a plausible alternative narrative available ("X must live in the other Claude," "the user must mean Y").

**3. Pattern-matching over evidence-weighting.** When a failure produces an unfamiliar signal (e.g., empty stderr with exit 255 from a wrapped SSH call), Claude's default response is to enumerate plausible causes from training data: generic SSH-troubleshooting checklists for ssh exit 255, generic MCP-environment-detection heuristics for missing tools. These enumerations skip the step of "what user-provided evidence is already in this conversation, and which candidates does it rule out?" The result is a list that looks comprehensive but is partly irrelevant to the actual problem.

These aren't bugs in any single model run. They're structural properties of how Claude reasons about its environment under uncertainty, and they fire predictably when MCP tools are in play because MCP introduces precisely the conditions that exercise them: partial visibility, deferred capabilities, and unfamiliar failure signatures.

## Diagnostic technique

For the user, or for a downstream Claude reading this entry:

1. **When Claude says "I don't have X tool," disbelieve until verified.** Have Claude run `tool_search` with relevant keywords. If the search returns nothing, the claim is supported. If it returns matching tools, Claude's prior was wrong and the tools are available. This single check would have prevented blind-spot instances 1 and 3 in the symptom section.

2. **When Claude enumerates possible causes, check whether existing conversational evidence rules any out.** If the user has reported behavior that already eliminates entries on Claude's list — e.g., "Termius works from the same laptop right now" rules out everything server-side — the list is partly noise. A single sentence usually corrects this: *"X has already ruled out causes A, B, C — focus on D and E."* Claude reasons well against explicit evidence pointers; it just sometimes doesn't generate them unprompted.

3. **When Claude reports its own runtime environment confidently, weight that against deterministic signals.** A popup the user can see, a log file on disk, a tool catalog dump — these are deterministic and trump Claude's introspection. If Claude says "I'm on claude.ai web" and the user is looking at Claude Desktop's title bar, the user is right by definition. Ask Claude to verify against a deterministic signal (read the log, call tool_search, etc.) before accepting any introspective claim.

## Canonical fix

There is no single canonical fix because the blind spots are structural rather than bugs. The fixes are procedural and live at the user / handoff-doc level:

- **Standing instruction for every Claude session interacting with MCP tools:** before answering "I don't have X tool" or "I can't do Y," call `tool_search` with relevant keywords. Treat `tool_search` as the first move when a question involves capabilities or actions, not the last.

- **Standing instruction for users debugging MCP issues with Claude:** Claude cannot see popups, log files, or browser dev-tools by default. When Claude's diagnosis seems off, paste the deterministic signals — popup screenshots, log file excerpts, MCP catalog dumps — into the conversation rather than describing them. Claude reasons much better against raw evidence than against narrated summaries.

- **Standing instruction for handoff docs:** when transitioning between Claude sessions in a multi-session arc, explicitly enumerate (a) what tools the receiving Claude should have access to, (b) what signals the receiving Claude cannot see directly, and (c) what user-provided evidence has already ruled out which candidate causes. This blunts all three blind-spot patterns at once.

## Operational consequence

The compounding cost of these blind spots is hours of unnecessary debugging per arc. In this arc specifically:

- The first instance (Claude misidentifying as web) cost one round of "let me explain why I think this is actually Claude Desktop" before returning to the popup as the diagnostic signal.
- The second instance (AWS-side enumeration) almost cost a user trip to the AWS console for an instance state that existing evidence had already ruled out. The user caught it before making the trip.
- The third instance (this Claude's own tool-surface miss) added an entire planned step — "paste this into Claude Desktop and ask it to commit via GitHub MCP" — that was unnecessary. The user caught it before the step was taken.

Each instance is cheap individually; the accumulating cost across an MCP-heavy debugging arc is meaningful. These are arcs that would most benefit from low-friction Claude collaboration, because they involve infrastructure setup that's already tedious without Claude getting in its own way.

A secondary cost worth naming: the user has to be *meta-aware* enough to override Claude's diagnoses with reference to evidence Claude can't see. This puts a cognitive load on the user that is exactly opposite to the load-reduction Claude is supposed to provide. Each individual override is small; the cumulative pattern erodes the user's default trust in Claude's diagnoses, which is its own cost.

## Recurrence prevention

- **Add to every SSP and handoff doc going forward:** *"Claude cannot see MCP attachment popups, log files, or browser-side UI. Trust deterministic signals (popups, logs, tool catalogs) over Claude's introspection. Call `tool_search` before claiming a tool is missing."* This is a one-line addition to the SSP template and meaningfully blunts all three blind-spot patterns at once.

- **For users:** when Claude's diagnosis feels off, the highest-leverage move is to paste the actual evidence (popup screenshots, log excerpts, command outputs) rather than re-narrating it. Claude pattern-matches better against concrete signal than against summary.

- **For receiving Claude sessions in handoffs:** before claiming the previous Claude must have been wrong about something, check whether the difference is "wrong" vs. "looking at different evidence." The Claude Desktop instance that said it had no SSH tools genuinely didn't have them — it just incorrectly inferred *why* from that fact. Similarly, the Claude that listed AWS causes wasn't lying about troubleshooting categories — it just failed to filter against given evidence. Diagnosing the *previous* Claude's reasoning failure mode is as important as solving the underlying technical problem.

- **For this Claude reading this PEL in a future session:** the third instance documented above occurred *while drafting PEL-009 and discussing PEL-011 by name*. The lesson was already in the conversation as a structural pattern; it still fired in the very session that was about to write it down. This is the strongest possible evidence that the pattern is structural rather than incidental — and the strongest possible argument for encoding the procedural fixes (always `tool_search` first, always weight existing evidence) as reflexes rather than relying on individual sessions to remember.

## Related context

This entry was logged in the same SSH MCP setup arc as PEL-009 (SSL inspection at install time) and PEL-010 (native-ssh-spawn issues at runtime). The three together comprise a complete map of one MCP install's lessons: install, runtime, and reasoning. PEL-011 is the meta-PEL — the previous two cover external failure modes; this one covers Claude's reasoning about external failure modes.

The strongest individual data point in this entry — instance 3, this Claude's own blind-spot moment — occurred near the end of the arc, during cleanup, in a session that had spent the previous several hours documenting and reasoning about exactly the kind of failure it was about to commit. That timing is the entry's most valuable feature: it demonstrates that procedural fixes (standing instructions, handoff-doc additions) are necessary even when the pattern is consciously known to all participants, because conscious knowledge of a pattern does not prevent a model run from re-instantiating it. The remedy lives in the workflow scaffolding, not in any individual model run's awareness.
