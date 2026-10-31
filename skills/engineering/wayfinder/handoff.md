# Interaction and handoff contract

Shared by wayfinder, to-spec, to-tickets, and matt-orchestrator. Apply this before choosing how to end a turn. **A turn ending is not a session completing; a session completing is not the effort completing.**

## Choose the state first

- **Waiting for you**: the current unit needs a human answer, approval, prototype reaction, or manual action. Present the decision and wait in this session. Partial answers leave the remaining questions pending. A recommendation is not the human's answer.
- **Working**: agent-runnable work remains in the current unit. Continue it; report progress only when useful.
- **Blocked**: no current work can proceed because of a dependency, another session's claim, missing access, or unavailable tracker state. Name the blocker and who can unblock it. Keep the unit incomplete.
- **Paused**: the human explicitly stops or requests a handoff before completion. Preserve progress, the exact unanswered question, and claim ownership on the tracker when available. Supply one resume instruction, clearly labelled incomplete.
- **Session complete**: this skill's completion gate passed, all required human input for the current unit was received and recorded, and its tracker writes succeeded. Only this state emits the completion block below.

Waiting takes precedence over completion. Before emitting a completion block, check both the conversation for unanswered questions and the tracker for unfinished work in the current unit. Artifact existence, an idle/done agent, a subagent report, and an empty frontier are not proof of completion. A failed publication remains incomplete; report the failure rather than a success handoff.

## Waiting turn

Use a compact prompt, not a closing block:

```text
WAITING FOR YOU · <skill or linked current ticket>
<The decision needed, with concise options and a recommendation when useful.>
<One question or precise manual action that unlocks the current unit.>
```

An interactive question tool can carry the options and question instead; avoid duplicating them in prose. End the turn at that question. Resume the same unit after the answer. Reserve next-skill suggestions, new-session paste blocks, and completion language for a completed unit or an explicit pause. Future HITL tickets can remain open after the current ticket resolves; unanswered questions required to resolve the current ticket cannot be moved to future work just to finish it.

## Readiness and ticket inventory

Before a completion block or orchestrator phase transition, refresh the relevant effort's map, spec, and **all** `wf-` and `i-` ticket states from the configured tracker. Scope to this effort, not the entire backlog. Use the tracker's lifecycle conventions, including explicit out-of-scope closures.

- **Way clear**: no open `wf-` tickets (including claimed, blocked, and waiting-for-human tickets), no in-scope fog, and no unanswered decision required by the destination. An empty frontier alone never qualifies.
- **Ready to implement**: the selected `i-` ticket is open, approved, unclaimed, has no unresolved blockers, and has the appropriate agent/human triage role. A ready label alone does not clear dependency edges, including `wf-` blockers.
- **Unknown**: if the tracker cannot be read, say which state is unknown. Do not report zero open tickets or recommend a transition on that basis.

Keep open-ticket lists in the output, not duplicated into the map. Link every ticket by its `wf-` or `i-` name. Show all open tickets once, grouped by kind; order ready/unclaimed, waiting for human, claimed/running, then blocked. Include type or triage role and blockers/owner where relevant. Claimed tickets are not on the frontier. A triage role alone is not a lifecycle closure.

## Completion block

Use plain uppercase labels and linked names; no decorative frame or table. Keep the outcome to one or two bullets. Count all open tickets, not just those created this session. For a kind with no open tickets, say `none`; if no implementation tickets have been created, say `not created`. Fog is separate from tickets. A completed planning session must not imply implementation has been delivered.

```text
SESSION RESULT · <charting complete | decision resolved | spec published | tickets published | planning complete>
- <What completed, with links to the map/spec/affected tickets.>

OPEN WAYFINDER · <N open | none> · <N frontier> · fog: <N patches | none>
- <linked wf-name> · <type> · <ready | waiting for you | claimed by owner | blocked by linked names>

OPEN IMPLEMENTATION · <N open | none | not created> · <N ready>
- <linked i-name> · <triage role> · <ready | waiting for you | claimed by owner | blocked by linked names>

NEXT ACTION
<One concrete action and why it is next; name its ticket/artifact and who acts.>
<One copyable command or instruction with the actual target path/URL, when useful.>

SESSION COMPLETE
```

The template describes output, not an outer code fence. Use one fenced `text` block for a copyable command when useful; replace placeholders with real paths/URLs rather than Markdown links inside the command. Skill prefix follows the target CLI: `/name` in Claude Code, `$name` in Codex; use the current harness's supported invocation elsewhere. Offer a new session only when it is useful or required, not when waiting for a reply here.

`SESSION COMPLETE` is the final standalone line, emitted once per successfully completed unit. It is a notification to refresh tracker state, never authorization by itself to close a pane or advance the workflow. Blocked, waiting, and paused turns do not emit it.

Choose **one** next action from actual state: resolve the next eligible decision, unblock a dependency, wait for its owner, graduate fog, synthesize the spec, approve/publish implementation slices, or implement a named ready ticket. Prefer continuing an existing eligible ticket over generating duplicates. If no work remains, state that; do not invent a follow-up. Each skill defines which transitions it owns.
