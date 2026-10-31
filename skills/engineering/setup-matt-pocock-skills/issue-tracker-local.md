# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Every effort carries a **three-letter code** (`ann`, `ber`, `edt`): lowercase inside ticket ids, uppercase in the plan's filename `.scratch/plans/<XXX>_<TITLE>_PLAN.md` and in its heading
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/i-<xxx>-<description>.md`, never a single combined tickets file. Wayfinder decision tickets use the same effort code under the `wf-` prefix: `.scratch/<feature-slug>/issues/wf-<xxx>-<description>.md`. The filename minus `.md` is the ticket **id**, and every reference to the ticket uses that id
- Dependency order lives in each ticket's `Blocked by:` line and in the plan's `## Ticket breakdown` list, not in the filenames
- Approved plans are autosaved to `.scratch/plans/` (`plan.autosaveDir` in `.omp/config.yml`); each ticket names its plan and the sections it covers in a `**Plan:**` line, and the plan lists the ids it was published as
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the ticket id directly.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `.scratch/<effort>/map.md` (the Notes / Decisions-so-far / Fog body).
- **Child ticket**: `.scratch/<effort>/issues/wf-<xxx>-<description>.md`, with the question in the body. A `Type:` line records the ticket type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved`.
- **Blocking**: a `Blocked by: wf-<xxx>-…, wf-<xxx>-…` line near the top. A ticket is unblocked when every ticket it lists is `resolved`.
- **Frontier**: scan `.scratch/<effort>/issues/` for tickets that are open, unblocked, and unclaimed; the order in the plan's `## Ticket breakdown` (or the map's list) decides between them.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
