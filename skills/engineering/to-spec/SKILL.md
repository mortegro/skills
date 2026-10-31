---
name: to-spec
description: "Turn the current conversation into a spec and publish it to the
  project issue tracker: no interview, just synthesis of what you've already
  discussed."
metadata:
  version: 0.1.0
  stage: alpha
disable-model-invocation: true
---

This skill synthesizes the current conversation, referenced map/spec, and codebase understanding into a spec. It does not run a new requirements interview; the targeted seam confirmation below is its only approval step.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run the `setup-matt-pocock-skills` skill.

Before starting, read [Interaction and handoff](../wayfinder/handoff.md). A seam question is **Waiting for you**, not a completed spec handoff.

## Process

1. Read any referenced source, including the wayfinder map and the resolved tickets relevant to the spec. Explore the repo if needed. Use the project's domain glossary vocabulary and respect relevant ADRs. Check that the spec's required decisions are actually settled. Missing decisions, open wayfinder tickets, or in-scope fog keep synthesis **Blocked**: name the gap and the decision work needed, rather than inventing an answer or publishing an agent-ready spec. Without a map, check the conversation for the same gaps.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations, unless the current seams were already explicitly accepted in the source conversation or recorded decisions. If confirmation is needed, end this turn with **Waiting for you** and resume here after the answer. Feedback or a proposed alternative is not yet acceptance; settle the revised seams before publishing.

3. Once the required decisions and seams are settled, write the spec using the template below and publish it to the project issue tracker. Record the seam decision and its human acceptance in Testing Decisions or a tracker comment. Apply the `ready-for-agent` triage label. The spec is not a ticket: do not give it a `wf-` or `i-` prefix. Any implementation tickets later derived from it use the `i-<xxx>-<description>` convention (`to-tickets` assigns the effort's three-letter code).

4. **Hand off only after publication succeeds.** Completion gate: required decisions settled, seams accepted, and the complete spec saved with a verifiable link. Refresh the effort's state and emit the shared completion block: **spec published**, both open-ticket inventories, and one **NEXT ACTION**. Normally this is `/to-tickets <actual spec path/URL>`. If existing `i-` tickets already cover the approved spec, name the next ready ticket and implementer instead of suggesting duplicate ticket generation. Newly discovered decision work takes precedence; name the `wf-` ticket/blocker rather than implying implementation readiness. A published spec is not completed implementation.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
