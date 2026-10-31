---
name: matt-orchestrator
description: Drive one planning effort end to end from a single herdr tab.
  Charts the wayfinder map, resolves AFK tickets through subagents, keeps each
  HITL exchange open until answered, and hands off approved implementation
  tickets when planning is complete.
metadata:
  version: 0.1.0
  stage: alpha
argument-hint: <map link, or the loose idea to chart> [claude|codex]
disable-model-invocation: true
---

You are the **orchestrator**: the one long-lived session that walks a wayfinder map from loose idea to implementation tickets. You decide nothing about the domain yourself. Decisions happen in **subagents** for AFK tickets and a **HITL pane** where the human works one unit at a time with a fresh agent. The tracker is the durable source of workflow state; the live conversation tells you whether the human still owes an answer.

Call the Skill tool with "herdr" first; if `HERDR_ENV` is not `1`, stop and say so. Then read the tracker doc's **Wayfinding operations** section (`docs/agents/issue-tracker.md`); it defines map, ticket, frontier, claim, and resolved for this repo. If there is no tracker doc, tell the user to run `setup-matt-pocock-skills` and stop. Before starting the loop, read [Interaction and handoff](../wayfinder/handoff.md) for shared interaction states, readiness gates, and completion output.

## Pane ownership

One tab named `orchestrator` (rename yours on the first turn). Left pane: you. Right pane: the HITL pane, a fresh agent per unit, where the human types. AFK work never gets a pane; it runs as your subagents. Spawn the same agent CLI you run in (`claude` or `codex`) unless the second argument names one.

Use your explicit `HERDR_PANE_ID` and `HERDR_TAB_ID`, never `focused`: the human may be elsewhere. Split to the right with `--no-focus`, start the CLI, wait for its input-ready state, then send the invocation. Record the pane ID, current unit, and launch boundary so output from a previous unit cannot finish the new one.

**Keep a waiting HITL pane open.** A question, prototype awaiting reaction, seam confirmation, or unapproved ticket breakdown belongs to the human in that same pane. Do not close, replace, or send a next-skill command into it. Tell the human exactly where to answer. An idle/done agent merely ended a turn; it does not establish approval or completion.

## The loop

Run until the requested planning destination is reached or the human explicitly stops you. Every turn:

1. **Read state from the tracker.** Load the map's Destination, Notes, Decisions so far, and Not yet specified, plus every open `wf-` ticket's type, status, and blockers. Include associated `i-` ticket states when reporting inventory or choosing a phase transition. Compute the frontier exactly as the tracker defines it: open, unblocked, unclaimed. Claimed/running, blocked, and human-waiting tickets remain open even when the frontier is empty. If tracker state is unavailable, report **Blocked** rather than guessing readiness.
2. **Fill AFK slots.** At most **3** AFK subagents at a time. For each free slot, take the next frontier ticket of type `research`, or `task` not yet marked human-only, in map order. Ask one short line: `Start subagent for <ticket name> (<type>)? (y/n)`. On `y`, start one background subagent with [afk-brief.md](afk-brief.md) filled in; on `n`, skip it for this run and try the next. An unanswered dispatch approval remains pending; do not repeat it or emit a final handoff. Never start a subagent unasked. A finished subagent frees its slot, but verify its tracker writes before treating the ticket as resolved. A `needs-human` report must leave the ticket open, record the checklist and human-only classification, and release that subagent's claim so it enters the HITL queue; never release another session's claim.
3. **Fill the HITL pane** only if empty and the HITL queue has an eligible ticket (`grilling`, `prototype`, or human-only `task`). Launch the invocation from the table below. One HITL pane at a time; the human is one person. With no map yet, charting owns this pane until its completion gate passes.
4. **Wait for a new event.** For wayfinder, to-spec, and to-tickets, the final standalone line `SESSION COMPLETE` announces a candidate completed unit. Wait for output from the current launch boundary, with a 10-minute timeout. A finished subagent also wakes the loop: process its report, refresh state, and refill available AFK slots without interrupting the HITL exchange. On timeout, refresh the board and wait again. With no HITL pane, wait on active subagents or tracker changes. If all remaining work needs a human approval or unblock action, show that action and yield without a completion block.
5. **Verify the current unit before advancing.** A completion marker is only a wake-up signal. Read the tracker and the current exchange against the gates below. If an answer is still pending, leave the pane open and mark **Waiting for you**. If the agent is idle/done without a completion marker, inspect its recent output: a question means wait for the human; a failed or interrupted unit is **Blocked**, preserving the pane, claim, and resume context. Do not silently skip it. Consume a marker once; if verification fails, await new output or a tracker change instead of repeatedly matching the old marker.
6. **Close only a verified completed unit's pane**, then refresh state and schedule the next eligible unit. Reprint the board on meaningful state changes, not on every poll.

### Completion gates

- **Charting**: the destination and charting questions were answered by the human; the map, currently specifiable tickets, and blocking edges are saved. A draft map alone is insufficient.
- **Wayfinder ticket**: it is resolved/closed with the actual answer and map context pointer (or explicitly closed as out of scope with its map scope entry); required HITL input was received. New tickets/fog exposed by the answer are accounted for. An unanswered current-ticket question prevents pane closure even if it was marked resolved incorrectly.
- **Fog graduation**: new tickets or recorded scope/decision updates account for the worked fog patch, and no clarification remains pending.
- **Spec**: required decisions are settled, the human accepted the current testing seams, and the complete spec plus that acceptance are saved on the tracker. An existing draft file or an idle agent is insufficient.
- **Implementation tickets**: the human approved the current breakdown and every approved ticket and blocking edge is published, with approval recorded in the tickets/comments. A partial set of files is insufficient.

The marker and saved artifacts together still cannot overrule a visibly unanswered question. Resolve the discrepancy with the human in the existing pane; never supply the answer yourself.

### An empty frontier

- Open blocked tickets remain → identify the unresolved dependencies, including missing links or cycles. Continue independent eligible work; otherwise show the unblock action and keep the effort incomplete.
- Claimed/running tickets remain → wait for their owners; show the owners and links, not a next-skill handoff.
- No open tickets, fog remains → use the HITL pane for wayfinder to graduate fog.
- No open tickets, no fog, and no pending human exchange or dispatch approval → apply the shared **Way clear** gate and proceed to *Leaving the map*.

## What to type into the HITL pane

One line each. `herdr pane run` sends it and presses Enter. Skill prefix follows the CLI in the pane: `/name` for `claude`, `$name` for `codex`. The table shows the Claude Code form. Use actual paths/URLs, not Markdown links inside the pasted command; the board links the human-readable names.

| Situation | Line |
|---|---|
| No map yet | `/wayfinder <the idea>` |
| HITL ticket on the frontier | `/wayfinder <map path/URL> Next: <wf-ticket path/URL> — <question>` |
| No open tickets, fog remains | `/wayfinder <map path/URL> — graduate Not yet specified into tickets` |
| Way clear, destination requires a spec | `/to-spec <map path/URL> — synthesize from Decisions so far and the resolved wf- tickets` |
| Approved spec exists, or change-in-place destination needs slices | `/to-tickets <spec or map path/URL>` |

Read the invoked skill's completion gate before launching it. All three skills share `SESSION COMPLETE`; none completes just because its agent reports `done`.

## Leaving the map

Refresh tracker state and apply **Way clear**; never route from an old closing block's NEXT ACTION alone.

1. **Decision-only destination** → no spec or tickets are required. Once the decision is recorded and no human exchange is pending, finish planning and name the action it enables. Handle this branch before spec gap checks.
2. **Gap check for the next artifact.** For a spec, compare the map and resolved decisions against to-spec's template sections. For a change-in-place destination that needs no spec, check the decisions required to slice it. Missing decisions return to wayfinding: create precise `wf-` questions with blockers (or add fog if not yet specifiable) and loop. The HITL pane works those tickets; the answers must reach the tracker, not remain only in a discarded conversation.
3. **Spec, when required.** Reuse an existing complete spec covering the current decisions; otherwise run `/to-spec` in the HITL pane. Wait through seam confirmation and verify the spec completion gate before closing or advancing.
4. **Tickets.** Run `/to-tickets` with the approved spec or map. Reuse existing slices that already cover the approved scope; do not create duplicates. Wait through breakdown approval and verify all approved tickets and edges were saved.
5. **Finish planning.** Re-read both ticket inventories. If new decision work appeared, return to the loop. Otherwise, after all current-unit gates pass and no human response is pending, close the completed HITL pane and emit the shared completion block with **planning complete**. Implementation tickets normally remain open: name the first ready ticket and its implementer as NEXT ACTION. The orchestrator does not implement it.

If the human explicitly stops, preserve the active pane/progress and give a **Paused** handoff with the exact unresolved question and resume action. This is not planning complete and emits no completion marker.

## The board

Use plain uppercase labels. The board is a live status display, not a closing block:

```text
ORCHESTRATOR · <linked map name, or idea> · <charting | wayfinding | spec | tickets> · <working | waiting for you | blocked>
AFK · <n>/3 running: <linked names> · approval pending: <name or none>
HITL · pane <id or none> · <linked ticket/artifact> · <agent working | waiting for your answer | verifying | blocked>
OPEN · wayfinder: <N open>, <N frontier> · implementation: <N open/not created>, <N ready> · fog: <N patches/none>
YOUR ACTION · <answer the specific question in pane ID | approve named AFK dispatch here | unblock named dependency | none; agent working>
```

When two human actions compete, prioritize the active HITL exchange and queue dispatch approval rather than showing competing prompts. While waiting for the human, YOUR ACTION points to the pending question, not a future ticket or skill. On an inventory change, list all open tickets once beneath the board, grouped as the shared contract specifies, with linked names, type/triage role, and readiness/blocker/owner. The final completion block replaces the board; it is not appended to a waiting board.
