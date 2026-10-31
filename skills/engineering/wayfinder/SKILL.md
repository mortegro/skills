---
name: wayfinder
description: Plan a huge chunk of work (more than one agent session can hold) as
  a shared map of decision tickets on your issue tracker, and resolve them one
  at a time until the way to the destination is clear.
metadata:
  version: 0.1.0
  stage: alpha
disable-model-invocation: true
---

A loose idea has arrived, too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. This skill charts the way as a **shared map** on the repo's issue tracker, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.

The destination varies per effort, and naming it is the first act of charting: it shapes every ticket. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place like a data-structure migration. The map is domain-agnostic: engineering work, course content, whatever fits the shape.

Before starting, read [Interaction and handoff](handoff.md). It defines waiting turns, completion blocks, and the shared open-ticket inventory.

## Plan, don't do

Wayfinder is **planning** by default: each ticket resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.

## Refer by name

Every map and ticket is an issue, so it has a **name**: its title. Name every Wayfinder ticket `wf-<short-descriptive-slug>`; reserve the `wf-` prefix for Wayfinder's decision tickets. In everything the human reads (narration, the map's Decisions-so-far), refer to it by that name, never by a bare id, number, or slug. A wall of `#42, #43, #44` is illegible; names read at a glance. The id and URL don't vanish; a name wraps its link, but they ride _inside_ the name, never stand in for it.

## The Map

The map is a single issue on this repo's issue tracker, labelled `wayfinder:map`, the canonical artifact. Its tickets are child issues of the map.

The map is an **index**, not a store. It lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place, its ticket, so the map never restates it, only gists it and links.

**Where the map, its child tickets, blocking, and frontier queries physically live is tracker-specific.** The issue tracker should have been provided to you. If not, tell the user to run the `setup-matt-pocock-skills` skill. Consult the tracker doc's "Wayfinding operations" section for how _this_ repo expresses them. If no tracker has been provided, default to the local-markdown tracker.

### The map body

The whole map at low resolution, loaded once per session. Open tickets are **not** listed: they are open child issues, found by query.

```markdown
## Destination

<what reaching the end of this map looks like: the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a ticket.>

## Notes

<domain; skills every session should consult; standing preferences for this effort>

## Decisions so far

<!-- the index: one line per closed ticket, enough to judge relevance, then zoom the link for the detail the ticket holds -->

- [<closed ticket title>](link): <one-line gist of the answer>

## Not yet specified

<!-- see "Fog of war": in-scope fog you can't ticket yet; graduates as the frontier advances -->

## Out of scope

<!-- see "Out of scope": work ruled beyond the destination; closed, never graduates -->
```

### Tickets

Each ticket is a **child issue** of the map, titled `wf-<short-descriptive-slug>`; the tracker's issue id is its identity. Its body is the question, sized to one 100K token agent session:

```markdown
## Question

<the decision or investigation this ticket resolves>
```

Each ticket carries a `wayfinder:<type>` label, one of `research`, `prototype`, `grilling`, `task` (see [Ticket Types](#ticket-types)).

A session **claims** a ticket by assigning it to the dev driving the map, **first**, before any work, so concurrent sessions skip it. That assignee _is_ the claim: an open, unassigned ticket is unclaimed.

Blocking uses the tracker's **native** dependency relationship: essential because it renders the frontier _visually_ in the tracker's own UI, so the human sees what's takeable without opening the map. Only a tracker that lacks native blocking falls back to a body convention. A ticket is **unblocked** when every ticket blocking it is closed; the **frontier** is the open, unblocked, unclaimed children, the edge of the known.

The answer isn't part of the body; it's recorded on resolution (see [Work through the map](#work-through-the-map)). Assets created while resolving a ticket are linked from the issue, not pasted in.

## Ticket Types

Every ticket is either **HITL** (human in the loop, worked _with_ a human who speaks for themselves) or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling agent that answers its own questions has broken this).

- **Research** (AFK): Reading documentation, third-party APIs, or local resources like knowledge bases to surface a fact a decision waits on. Resolved by a subagent that calls the Skill tool with "research". Use when knowledge outside the current working directory is required.
- **Prototype** (HITL): Raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to (an outline, a rough take, a stub, or UI/logic code) by calling the Skill tool with "prototype". Links the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Grilling** (HITL): Conversation. The default case. Always call the Skill tool twice, for "grilling" and "domain-modeling".
- **Task** (HITL or AFK): Manual work that must happen before a _decision_ can be made: nothing to decide, prototype, or research, but the discussion is blocked until it's done. Signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. This is the one type that _does_ rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). Resolved when the work is done; the answer records what was done and any resulting facts (credentials location, new URLs, row counts) later tickets depend on.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the **fog of war**: the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, one at a time, until the way to the destination is clear and no tickets remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It's the undiscovered frontier _toward_ the destination: everything here is in scope, just not sharp enough to ticket. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators reading where the effort is headed.

**Fog or ticket?** The test is whether you can state the question precisely now, _not_ whether you can answer it now.

- **Ticket when** the question is already sharp, even if it's blocked and you can't act on it yet.
- **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into ticket-sized pieces: it's coarser than a ticket, and one patch may graduate into several tickets, or none, once the frontier reaches it.

**Not yet specified** excludes what's already decided (Decisions so far), what's already a live ticket, and what's out of scope (the next section).

## Out of scope

Fog only ever gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope**: it isn't fog, and it doesn't belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work you've consciously ruled out of _this_ effort. Scope, not sharpness, lands it here.

Out-of-scope work never graduates (the frontier stops at the destination), so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.

Ruling something out of scope is a scoping act, not a step on the route. When a ticket that already exists turns out to sit past the destination (mis-scoped in while charting, or exposed by a resolution), **close it** (a closed ticket is unambiguously off the frontier) and leave one line in the **Out of scope** section: the gist plus why it's out of scope, linking the closed ticket. It stays out of **Decisions so far**, which records the route actually walked; a scope boundary isn't a step on it.

## Session boundaries

Complete only the unit selected by the invocation:

- **Charting**: the human settled the destination and answered the breadth-first questions; the map and currently specifiable tickets are saved with blocking edges. Questions still awaiting the human keep charting in **Waiting for you**, even if draft artifacts already exist.
- **Working a ticket**: its question is fully answered, required HITL input actually came from the human, the resolution is saved, the ticket is closed/resolved, and the map and newly exposed fog/tickets are updated. An explicit out-of-scope closure uses the recorded scope boundary and map's Out of scope entry instead of a resolution/decision pointer. A prototype shown, a recommendation made, or a checklist handed to the human is progress, not resolution.
- **Graduating fog**: when no ticket is available, sharpen fog with the human until it becomes saved tickets or is accounted for by recorded decisions/scope boundaries. An unanswered clarification keeps the session waiting.

While a required question is unanswered, use the shared **Waiting for you** prompt and resume here after the reply. The one-ticket-per-session limit counts resolved tickets, not turns or questions. Keep asking within that ticket until its completion gate passes. If the human explicitly pauses, preserve the unresolved question and claim for resumption; do not call the ticket resolved.

After the unit completes, use the shared completion block with both open `wf-` and `i-` inventories. Choose its **NEXT ACTION** from refreshed state:

1. Open `wf-` tickets remain → continue `/wayfinder` on a named frontier ticket. If none is eligible, name the blocking dependency or claim owner and the unblock/wait action; do not select a blocked or claimed ticket as ready.
2. No open `wf-` tickets, but fog remains → `/wayfinder` on the map to graduate the next fog patch.
3. **Way clear**, destination is an unwritten spec → `/to-spec` on the map.
4. **Way clear**, approved spec exists or change-in-place destination is ready to slice → `/to-tickets` on the spec/map, unless implementation tickets already cover that work; then name the next ready `i-` ticket and its implementer instead.
5. **Way clear**, destination was a decision → name the concrete action it enables, or state that the requested decision is recorded and no further work was requested.

Charting can complete while future decision tickets remain open; say **charting complete**, not **way clear**. The map is complete only under the shared **Way clear** gate, which checks all open tickets, not just the frontier.

## Invocation

Two modes. Either way, **never resolve more than one ticket per session**, with the exception of research tickets.

### Chart the map

User invokes with a loose idea.

1. **Name the destination.** Call the Skill tool twice, for "grilling" and "domain-modeling", to pin down what this map is finding its way to: the spec, decision, or change. The destination fixes the scope, so it's settled first.
2. **Map the frontier.** Grill again, **breadth-first** this time: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. Wait for answers to this charting exchange before creating the final map. **If this surfaces no fog** (the way to the destination is already clear, the whole journey small enough for one session), you don't need a map. Ask the user how they'd like to proceed using **Waiting for you**; this question is not a completed handoff.
3. **Create the map** (label `wayfinder:map`): Destination and Notes filled in, Decisions-so-far empty, the fog sketched into **Not yet specified**.
4. **Create the tickets you can specify now** as child issues of the map, then wire blocking edges in a **second pass** (issues need ids before they can reference each other). Wiring sorts them into the frontier and the blocked; everything you can't yet specify stays in the fog: the **Not yet specified** section.
5. **Fire eligible research subagents.** Dispatch only unblocked, unclaimed `research` tickets. Each subagent calls the Skill tool with "research", capturing its findings on a throwaway `research/<name>` branch with a context pointer from the ticket. Running/claimed research is still open and is not on the frontier.
6. **Complete charting** only after its [session boundary](#session-boundaries) is satisfied; charting hand-resolves nothing. Emit the shared completion block, naming the next eligible decision or the dependency/owner to wait for.

### Work through the map

User invokes with a map (path, URL, or number). A ticket is **optional**: without one, you pick the next decision, not the user.

1. Load the **map** and ticket states: the low-res view, not every ticket body.
2. Choose the ticket. If the user named one, verify it is open, unblocked, and available to this session; resume your own active claim, but never take another session's claim. Otherwise take the first frontier ticket in order. **Claim it** per the tracker before any work. If none is available, graduate remaining fog with the human when no tickets are open; otherwise report the dependency/claim blocking progress. Never infer map completion from an empty frontier.
3. Resolve it. **Zoom as needed**: fetch the full body of any related or closed ticket on demand; call the Skill tool for whichever skills the `## Notes` block names. If in doubt, call the Skill tool twice, for "grilling" and "domain-modeling". For HITL, ask and wait for the human; after partial answers, continue with the remaining question in this same session. Advance only when the ticket's question is fully answered or its manual action is confirmed done.
4. Record the resolution according to the tracker conventions: save the answer, **close/resolve** the issue, and **append a context pointer** to the map's Decisions-so-far. Include the human's actual decision for HITL, not an unaccepted recommendation.
5. Add newly-surfaced tickets (create-then-wire); graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its new ticket. If the answer reveals that a ticket (this one or another) sits beyond the destination, **rule it out of scope** rather than resolving it on the route. If the decision invalidates other parts of the map, update or delete those tickets.
6. **Complete the session** only after its [session boundary](#session-boundaries) is satisfied. Refresh state and emit the shared completion block; its next action continues wayfinding or leaves the map only when the corresponding readiness gate passes.

The user may run unblocked tickets in parallel, so expect other sessions to be editing the tracker concurrently.
