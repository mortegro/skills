---
name: implement
description: Implement a spec or a published ticket set, working the ready
  frontier wave by wave until every ticket lands or a blocker needs the human.
metadata:
  version: 0.1.0
  stage: alpha
disable-model-invocation: true
---

Implement the work named in the argument (plan path, effort code, spec, or ticket list). Orchestrate: subagents build the tickets; you verify and land them.

Tickets named `i-<xxx>-<description>` are implementation tickets. Decision tickets (`wf-…`) are resolved through the `wayfinder` skill, unless the user explicitly asks to execute the decision.

## 1. Load the work

- A plan (`.scratch/plans/<XXX>_<TITLE>_PLAN.md` or `.omp/plans/`; an effort code resolves to its plan): read its `## Ticket breakdown` for dependency order, then every listed ticket: body, `**Blocked by:**`, and the plan sections its `**Plan:**` backlink cites.
- Read `docs/agents/issue-tracker.md` ("Wayfinding operations") when present: it defines claim, resolve, and frontier for this repo. Real trackers use native blocking links.
- A spec or request with no tickets: implement it directly in this session, single-agent, using the practices in step 3's brief, then continue at step 7.

Done when every ticket in the breakdown is read and its blockers known.

## 2. Compute the frontier

Frontier = open, approved, unclaimed tickets whose blockers are all done. `wf-` tickets on the frontier are resolved via `wayfinder`; leave them to the human when they need a decision (step 6).

Done when you hold the list of tickets for this wave. An empty frontier with open tickets left means jump to step 7.

## 3. Run a wave

Claim each frontier ticket per the tracker conventions. Dispatch one subagent per ticket, in parallel (omp `task`, Claude Code Agent/Task tool, Codex's subagent equivalent). Tickets whose changes would touch the same files run in separate waves.

Brief each subagent self-contained:
- the ticket body and the plan sections it cites;
- /tdd at pre-agreed seams;
- typecheck and its own single test files, run regularly;
- commits, the full suite, and formatters stay with the orchestrator.

Done when every subagent in the wave has reported.

## 4. Verify the wave

Run typecheck and the affected tests yourself. On red, dispatch a corrective subagent naming the gap, then re-verify.

Done when the wave is green; the next wave starts only then.

## 5. Land

Mark each ticket resolved per the tracker doc (no doc: note the resolution on the ticket), then commit to the current branch, one commit per ticket, staging the ticket's code together with its resolved ticket file: `<XXX>: <ticket title> (i-<xxx>-…)`. Recompute the frontier and return to step 3.

Done when each green ticket has its own commit that carries both its code and its resolved mark, and the working tree holds no leftover ticket changes.

## 6. Park blocked tickets

A ticket that needs a human decision stays open with the reason and the owner recorded on it. Independent tickets keep going.

Done when the reason is on the ticket and the frontier excludes it.

## 7. Finish

Run the full test suite once. Run /code-review over the effort's commits on every effort, whatever its size; fix the findings and commit the fixes.

Final report:
- tickets landed, each with its commit;
- tickets still open, each with its blocker and owner.

Done when the full suite is green, /code-review has run over the effort's commits with its findings fixed and committed, and every ticket has landed or every remaining one is blocked on a named human action.
