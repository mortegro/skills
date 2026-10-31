---
name: matt-status
description: Show every map's decision and implementation status, with
  ready-to-take frontiers; pass detailed to list every ticket, including
  resolved and closed tickets.
metadata:
  version: 0.1.0
  stage: alpha
argument-hint: "[detailed]"
disable-model-invocation: true
---

# Matt Status

A **read-only snapshot** of all wayfinder maps and their implementation work in the current repository's configured tracker. Report what is recorded, not what you assume has been built. No herdr session is required.

## Invocation

- `/matt-status` — overview of every map, progress counts, and the complete decision and implementation frontiers.
- `/matt-status detailed` — the same map coverage and progress, expanded to **every** decision and implementation ticket, including completed, resolved, cancelled, and out-of-scope tickets.

Use `$matt-status` in Codex and the supported skill prefix in other harnesses. Trim the argument; accept only no argument or `detailed`. For anything else, show these two forms and stop rather than silently choosing a mode. Both modes cover all maps; `detailed` changes depth, not scope.

## Read the tracker

1. Read the repository's configured issue-tracker and triage-label docs, including **Wayfinding operations**, and each map's tracker overrides in Notes. If configuration is missing or unreadable, report that limitation; do not create setup files or guess a tracker. Read [the shared readiness and ticket inventory rules](../wayfinder/handoff.md#readiness-and-ticket-inventory). Only those state/readiness rules apply here: this skill emits a status snapshot, not a workflow completion block or `SESSION COMPLETE` marker.
2. Discover **all maps**, including closed/completed maps. For the local-markdown tracker, enumerate `.scratch/**/map.md`, then each effort's spec and all `issues/wf-*.md` and `issues/i-*.md` files. Include ignored `.scratch` content. For a remote tracker, query `wayfinder:map` in this repository/project across all lifecycle states, follow all result pages, and fetch all children and associated specs/implementation issues across all states. Use the configured relationships, not an open-only query.
3. Associate implementation tickets through their explicit map/spec/parent links, or the same effort directory locally. A spec is an artifact, not an implementation ticket. Shared tickets may appear under each linked map, but count each identity only once in repository totals. If implementation tickets have no discoverable map, list them separately under **UNMAPPED IMPLEMENTATION**, with the same frontier/detail rules; do not hide them or invent a map association.
4. Read enough of each map, spec, and ticket to establish the destination, fog, lifecycle, type/triage role, claim/owner, blockers, and recorded approval/completion. Read relevant comments or answers when metadata alone is insufficient. Accept the configured Markdown metadata form, including plain `Status:` and bold `**Status:**`; do not assume a new schema. Follow dependency references, including cross-map and `wf-` → `i-` edges. Resolve identities using full paths/URLs or tracker IDs so repeated slugs in different efforts cannot collide.
5. Compute the snapshot before formatting it. Use tracker order (`Order:`, issue number, or the configured ordering), with title/path as a stable fallback. Sort maps by name/path. If a query is truncated, page it; if a file/issue cannot be read, keep the affected map/ticket visible as **unknown** and name the missing source. Counts with incomplete coverage are lower bounds or unknown, never exact zeros. Do not infer an empty tracker from a failed lookup.

These are reads only: never claim, resolve, create, relabel, or repair tickets; launch agents or panes; invoke planning/implementation skills; or run builds/tests to investigate completion. Surface inconsistent or missing state as a finding with links.

## Classify without overstating progress

Keep **lifecycle**, **readiness**, and **human involvement** separate:

- Lifecycle comes from the tracker: open/in progress, resolved/completed, or closed without delivery (cancelled/out of scope/wontfix). A triage label such as `ready-for-agent` or `ready-for-human` is not completion. Checked acceptance boxes, a spec file, a branch, or a claim alone do not prove delivery. A closed ticket with no completion reason is **closed, outcome unknown**, not completed.
- Claims indicate ownership, not evidence a worker is currently running. Show the recorded owner/worker; use **running** only when recorded activity supports it. Use wayfinder's [Ticket Types](../wayfinder/SKILL.md#ticket-types) for HITL/AFK classification; Task participation depends on its recorded manual/agent requirements. A HITL type means human participation is required; call it **waiting for your answer** only when an actual pending question/approval/manual action is recorded or visible in the current conversation.
- A frontier contains only open, unclaimed tickets whose blockers are satisfied under the configured tracker rules. Implementation additionally requires approved scope and the correct agent/human triage role, per the shared readiness gate. Missing blockers, cycles, unresolved decisions, or unknown approval keep a ticket off the ready frontier; state the reason. Do not treat cancellation/out-of-scope closure as a satisfied dependency unless the tracker explicitly does so.
- Use the shared **Way clear** gate for map planning readiness. Claimed/blocked tickets and fog still count even when the frontier is empty. A map labelled closed with remaining decisions is inconsistent, not clear. A map may be planning-clear while implementation has not started or is still active.
- Show implementation as **not created**, **not started**, **in progress**, **blocked**, **complete (tracker-reported)**, **closed without full delivery**, **mixed**, or **unknown**, whichever the recorded states support. Preserve separate completed/open/closed-without-delivery counts; do not report a completion percentage from ticket counts or present tracker-reported completion as runtime verification. **Planning-only describes wayfinder's execution scope, not the absence of downstream implementation.** For a map preparing a later build, no `i-` tickets means **not created**, even when its Out of scope excludes building during wayfinding. Use **not required by this map** only for a decision-only destination with no planned downstream build; still show any associated implementation tickets that exist.
- Spec state is separate: link it and report **draft**, **published**, **unknown**, **not created**, or **not required** according to the recorded evidence/destination. File existence alone proves presence, not approval or publication.

Counts must reconcile. For each ticket kind, total = open + completed/resolved + closed without completion + unknown lifecycle. Frontier, claimed, blocked, and waiting-for-human counts are subsets of open, not extra lifecycle totals; they may overlap. Keep fog separate from tickets, counting recorded patches rather than inventing ticket-sized questions.

## Default overview

Use short headings and linked ticket names, not a wide terminal table. Begin with **MATT STATUS · overview**, the repository/project, and map count. Include one block for **every** map:

```text
<Linked map name> · planning: <active | blocked | clear | unknown>
Destination: <one-line gist>
Wayfinder: <resolved>/<total> resolved · <N> open · <N> closed without resolution · <N> unknown · <N> frontier · fog: <N patches | none | unknown>
Spec: <linked artifact and state | not created | not required | unknown>
Implementation: <state> · <completed>/<total> completed · <N> open · <N> closed without delivery · <N> unknown · <N> ready

DECISION FRONTIER
- <linked wf-name> · <type, HITL/AFK> · <one-line question>
IMPLEMENTATION FRONTIER
- <linked i-name> · <agent/human> · <one-line deliverable>
ATTENTION: <pending human question + ticket, claims + owners, unresolved blockers, or unknown state; omit if none>
```

Render the template as normal prose, not an outer code fence. For **not created/not required**, omit meaningless implementation fractions. List **every frontier ticket**, in tracker order. For an empty frontier, say `none` and why (claimed, blocked, fog-only, no tickets yet, or all resolved); never equate it with completion. Keep non-frontier tickets to counts and concise attention notes in overview, rather than expanding the entire backlog. Explicitly distinguish claimed and blocked work from ready tickets.

End with one **NEXT ACTION** grounded in this snapshot: answer a recorded pending question, take a named frontier ticket, wait for its owner, unblock a named dependency, or graduate fog. If planning is clear and the destination calls for the next artifact, name the spec/ticket step; if everything requested is complete, say so. Give a concrete command with the real target path/URL when useful, but do not execute it or ask the question on another session's behalf. Add the one-line hint `Use /matt-status detailed for all tickets` with the correct CLI prefix.

## Detailed overview

Keep the same per-map summaries and final next action. Replace the two frontier lists with **WAYFINDER TICKETS** and **IMPLEMENTATION TICKETS** listing **all tickets once per associated map**, not just open tickets. Mark frontier/ready membership inline so readiness stays visible without repeating tickets in separate frontier lists.

Each ticket gets a compact bullet or, when needed, two lines:

- **Linked name** — one-line question/deliverable; type and HITL/AFK or implementation triage role; recorded lifecycle; **frontier/ready**, **claimed**, **waiting for your answer**, **blocked**, **needs approval/triage**, or **unknown**, as applicable.
- Owner/worker if claimed; unresolved blockers by linked name (missing references identified explicitly); exact pending human question/action when known. For resolved/completed/closed tickets, show the recorded outcome/reason in one line when available, preserving out-of-scope/cancelled distinctions. Keep full answers and comment histories behind their links.

Include fog patches as short bullets and surface dependency cycles, broken references, or conflicting status evidence beside the affected tickets. If a readable inventory is large, use successive sections/messages until every ticket is covered; never sample, silently truncate, or replace the rest with “and N more.” If tracker access prevents completeness, label the report **partial** and enumerate what could not be read. Omit the `detailed` hint in this mode.

## Finish the snapshot

Before responding, ensure every discovered map appears; each ticket is accounted for in counts; every eligible frontier ticket is shown; and in detailed mode every ticket has its own entry, including closed tickets. With no maps or tickets after a successful complete discovery, say **No maps or implementation tickets found**. With only unmapped implementation, show that group. Report status without a planning-session completion marker or any tracker mutations.
