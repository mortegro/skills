# AFK subagent brief

Fill the angle brackets and send as the subagent's whole prompt. One ticket per subagent. Run them in the background, in parallel.

```text
You are resolving ONE wayfinder decision ticket alone (AFK). Decide or find out; do not build the destination.

Tracker conventions: read the "Wayfinding operations" section of <tracker doc path> first. It defines how to claim, resolve, and where Decisions so far live.
Map: <map name> at <map link>. Read its Destination, Notes, and Decisions so far before anything else.
Ticket: <ticket name> at <ticket link>. Type: <research | task>.

Steps:
1. Claim the ticket per the tracker conventions, before any work. If it is already claimed, stop and report "claimed".
2. Resolve it:
   - research: call the Skill tool with "research". Capture findings on a throwaway branch research/<ticket name> and link it from the ticket.
   - task: do the work. If a step needs the human (a login, a purchase, a decision), keep the ticket open, record "human-only" and the precise checklist on the ticket, release only your own claim per tracker conventions, and report "needs-human". Stop here: no resolution, map decision pointer, or completion claim until the human has done the action.
3. Record: post the answer as the ticket's resolution (the ## Answer heading or resolution comment the tracker doc names), mark it resolved, and append one line to the map's Decisions so far: [<ticket name>](<ticket link>): <one-line gist>.
4. If the answer sharpens any patch in Not yet specified into a precise question, create that ticket as a child of the map with its blockers, and remove the patch from Not yet specified. Do not pre-slice fog you cannot phrase precisely.
5. Do not resolve any other ticket.

Report in exactly this shape, nothing else:
STATUS: resolved | needs-human | claimed | blocked
GIST: <one line>
NEW TICKETS: <names, or none>
CHECKLIST: <only for needs-human>
```
