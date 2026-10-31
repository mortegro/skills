---
name: to-tickets
description: Break a plan, spec, or the current conversation into a set of
  tracer-bullet tickets, each declaring its blocking edges, published to the
  configured tracker (edges as text in one file per ticket locally, or native
  blocking links on a real tracker).
metadata:
  version: 0.1.0
  stage: alpha
disable-model-invocation: true
---

# To Tickets

Break a plan, spec, or conversation into a set of **implementation tickets**: tracer-bullet vertical slices, each declaring the tickets that **block** it. Name every ticket `i-<xxx>-<description>` — `<xxx>` is the effort's three-letter code (see step 5) and `<description>` is two to four kebab-case words; reserve the `i-` prefix for implementation tickets.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run the `setup-matt-pocock-skills` skill.

Before starting, read [Interaction and handoff](../wayfinder/handoff.md). Use its waiting state for breakdown approval when step 4 requires one, and its completion block only after publication succeeds.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.

Read existing `wf-` and `i-` tickets for this effort before drafting. Reuse existing implementation slices rather than creating duplicates. If an associated wayfinder map still has open decisions or fog required by this work, remain **Blocked** and name the decision to resolve; do not turn unanswered product questions into agent-ready implementation tickets. With no map, use the same check against unresolved decisions in the conversation/spec.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches are too large for one context window, split by blast radius further; the expand/contract boundary itself never moves.

### 4. Approve the breakdown

**Source is an approved plan** (the plan-mode handoff): the plan's approval already covers its `## Ticket breakdown`, so do not ask again. Skip the quiz and go straight to step 5. Follow the plan's breakdown, or draft one from the plan when it has none; where a slice has to be split, merged, or re-edged to satisfy the vertical-slice rules, make that change yourself and list each deviation in the completion block's outcome bullet. Only an unresolved product decision (step 1) stops publication.

**Any other source** (spec, issue, conversation): present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: `i-<xxx>-<description>`
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work

Ask one approval question covering granularity, genuine blocking edges, and any merge/split requests. End with **Waiting for you**, not a completion block or a suggestion to start implementing.

Iterate here until the user explicitly approves the current breakdown. Feedback on one slice is not approval of the others; a revised breakdown needs approval unless the user already authorized those exact changes. Publish only after this gate passes.

### 5. Give the effort a three-letter code

Before anything is published, the effort gets a **three-letter code**: lowercase inside ticket ids, uppercase in the plan's filename and heading. It is the short handle that names this set of tickets in filenames, conversation, commit messages, and branch names, so it has to be stable and unique.

- Derive it from the effort's own words — initials or the first distinctive consonants (`annotation inbox` → `ann`, `Bereiche/Bibliothek` → `ber`, `editor structure` → `edt`). Letters only, never digits.
- Check it is free before using it: the codes already taken are the prefixes of `.scratch/plans/*_PLAN.md` and of the ticket ids in `.scratch/*/issues/i-*.md` and `wf-*.md`. On a collision pick different letters; never disambiguate with a number.
- Rename the plan file to `.scratch/plans/<XXX>_<TITLE>_PLAN.md` and prefix its `#` heading with `<XXX> — `. omp derives the autosave filename from that heading (uppercased, non-alphanumerics collapsed to `_`, suffixed `_PLAN.md`), so a later re-approval of the same plan saves under the same name instead of a second file.

### 6. Publish the tickets to the configured tracker

Publish the approved tickets. **How** depends on the tracker `setup-matt-pocock-skills` configured; the tickets are the same either way, only the shape of the blocking edges changes:

- **Local files** → write one file per ticket at `.scratch/<feature-slug>/issues/i-<xxx>-<description>.md`, where `<xxx>` is the effort code and `<description>` is two to four kebab-case words naming what the ticket delivers (`i-ann-inbox-contract.md`, `i-edt-preview-pick.md`). That filename minus `.md` is the ticket's **id**, and the id is how every other document refers to it — blocking edges, the plan's breakdown, the conversation. Create them in dependency order (blockers first). Filenames do not sort into dependency order, so the order lives in each ticket's "Blocked by" and in the plan's `## Ticket breakdown`. Use the per-ticket file template below: one ticket per file, never a single combined file.
- **A real issue tracker (GitHub, Linear, …)** → publish one issue per ticket in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers. Title each issue `<XXX>: <Ticket title>` so the effort code survives where there are no filenames. Use the platform's native blocking / sub-issue relationship where it has one; otherwise set each ticket's "Blocked by" to the blocking issues. Apply the `ready-for-agent` triage label unless instructed otherwise; the tickets are agent-grabbable by construction.

The implementation **frontier** is the open, approved, unclaimed tickets whose blockers are all done; the plan's `## Ticket breakdown` list (or each ticket's `Blocked by`) gives the order when several are free at once, since filenames don't. Report the frontier at handoff; this skill creates tickets, it does not start implementing them.

Do NOT close or modify any parent issue.

### 7. Link the tickets and the plan to each other

If the source was an approved plan, that plan file stays the ticket set's source of truth, and the link has to survive the session that wrote it. Make it hold in both directions:

- **Ticket → plan**: every published ticket carries a `**Plan:**` line naming the plan's repo-relative path and the sections of it that this ticket covers.
- **Plan → tickets**: once the tickets exist, append their ids (or issue references) to the plan's `## Ticket breakdown` section, one line per ticket, in the same order as the breakdown. This list is what carries the dependency order that the filenames no longer encode.

Take the plan's path from the directory plans are saved in, not from the approval message. Under omp the approval message names a session-local `local://<slug>-plan.md` copy that disappears with the session; the durable copy is autosaved to `plan.autosaveDir` (in this repo: `.scratch/plans/`, alongside the tracker). The two files have identical content, so cite the durable one.

<local-ticket-template>

# i-<xxx>-<description>: <Ticket title>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.

**Plan:** the plan file's repo-relative path and the sections this ticket covers, or omit this line when the tickets came from something other than a plan.

**Blocked by:** the ids of the tickets that gate this one (`i-<xxx>-<description>`, comma-separated), or "None (can start immediately)".

**Status:** ready-for-agent

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

</local-ticket-template>

<issue-template>

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## Plan

The plan file's repo-relative path and the sections this ticket covers (omit when the tickets came from something other than a plan).

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to each blocking ticket, or "None (can start immediately)".

</issue-template>

In either form, avoid specific file paths or code snippets: they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits. The `**Plan:**` backlink is the other exception: it is a path on purpose, and it stays valid because the plan file is written once and lives in the repo.

### 8. Hand off implementation

Completion gate: the breakdown is approved (by the human in step 4, or by the plan approval when the source was an approved plan), every ticket and blocking edge is saved on the configured tracker (including the `**Plan:**`/`## Ticket breakdown` backlinks), and no publication remains pending. Record how the breakdown was approved in the tickets or their comments so another session can verify it. If only some writes succeed, report the saved links and remaining publication work as incomplete; resume without duplicating those tickets.

Refresh the effort's state and emit the shared completion block: **tickets published**, both open-ticket inventories, and one **NEXT ACTION**. Implementation runs in a **fresh session** that applies the `implement` skill to the whole published ticket set and works the frontier wave by wave, so this session stops at publication. NEXT ACTION names the first ready `i-` ticket and what it delivers, so the human sees where work starts, then gives the copyable command for the current harness, filled with the effort's actual code and the repo-relative durable plan path from step 7:

- **omp** → `/implement <XXX>`
- **Claude Code** → start a fresh session (`/clear`), then `/implement <plan path>`
- **Codex** → start a fresh session, then `$implement <plan path>`

If all implementation tickets are blocked or claimed, name the blocker/owner and the action needed instead. Ticket publication is complete planning output, not completed implementation.
