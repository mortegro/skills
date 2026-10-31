---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use
  when the user wants to stress-test their thinking, or uses any 'grill' trigger
  phrases.
metadata:
  version: 0.1.0
  stage: alpha
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round through the `ask` tool, then wait for the user's answers before the next round.

Put a round to the user like so:

- **Grounding in chat, same response as the `ask` call.** For each question, numbered `Q1`, `Q2`, … in frontier order, write the facts, evidence and tradeoffs the user needs to decide. The `ask` dialog carries only the decision itself.
- **One `ask` call per round**, one entry in `questions` per frontier question: `id` `q1`, `q2`, …; `header` a short title chip; `question` the decision in one sentence.
- **Recommendation first.** Option 1 is always your recommended answer, with `recommended: 0`; the alternatives follow it. Each `label` names the choice in a few words; its `description` carries the tradeoff and, on option 1, why you recommend it. The tool appends "Other" itself, so list only real choices.
- `multi: true` only when the choices genuinely combine.

```json
{
  "id": "q1",
  "header": "Q1 result contract",
  "question": "What does a successful answer promise the caller?",
  "recommended": 0,
  "options": [
    { "label": "Effective next turn", "description": "Recommended: matches what the consumer already relies on; no new wire state." },
    { "label": "Accepted only", "description": "Today's contract; the consumer must keep its fallback." }
  ]
}
```

Without an `ask` tool, write the same round in chat: numbered questions, each listing your recommended answer first.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
