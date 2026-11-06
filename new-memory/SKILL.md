---
name: new-memory
disable-model-invocation: false
description: Record a new memory card in the central memory vault, under the current project's subfolder. Use when the user says "remember that ...", "note that ...", "add to memory", or after finishing a piece of work worth keeping. Creates one linked card and registers it in the project's MEMORY.md.
---

# new-memory

Create ONE focused linked memory card in the current project's memory subfolder and
register it in that subfolder's `MEMORY.md`.

The vault is central: `/scratch/ap7151/MEMORY/`, with a systemwide hub (`MEMORY.md`)
linking per-project subfolders (`slt/`, `macro-micro/`, `ssl-codebook/`, ...), each
with its own `MEMORY.md`.

## Steps

### 1. Locate the vault and the project subfolder

The vault is always `/scratch/ap7151/MEMORY/`. Determine the current project from the
cwd: list `ls /scratch/ap7151/MEMORY/` and match the cwd (project directories live at
`/scratch/ap7151/<PROJECT>/`) against the subfolder names — the names may differ slightly
(e.g. `MICRO_TO_MACRO` vs `macro-micro`), so match loosely or ask the user. If no
subfolder matches the current project, offer to run `init-memory` to create one — never
create cards without a hub (an unregistered card is lost).

### 2. Check the vault first (no duplicates)

`grep -ri "<topic>" /scratch/ap7151/MEMORY/ --include="*.md"` and grep the project hub
too. If the fact already exists, update the existing card instead of creating a second one.

### 3. Pick a card type from the filename

- `project_<slug>.md` — topic, workstream, experiment lineage
- `feedback_<slug>.md` — lesson / do-don't / preference
- `reference_<slug>.md` — reusable recipe / how-to / key facts / script map
- `user_<name>.md` — person profile
- `session-<NNN>-<slug>.md` — chronological log entry (append, don't overwrite)

One topic per card. If it has no clear name, it's two cards.

### 4. Write ONE focused card

Write the content as plain, natural, relaxed prose — like a person explaining what
happened to another person, not an incident report. Headings are fine for
organization, but the text under them should be normal sentences.

- No dense, compressed technical prose like "failure → root cause → fix → requeued"
  unless the user explicitly asks for a postmortem or formal report.
- Avoid labels like "The failure:", "Root cause:", "The fix:", "Current state:" by
  default. Just tell what happened naturally.
- Don't compress several technical facts into one long sentence. Important thing
  first, details only when they help.
- Technical terms are fine when they're actually useful; skip jargon and internal
  shorthand.

```markdown
# <Topic>
**Created:** YYYY-MM-DD
**Last updated:** YYYY-MM-DD

## Context
<!-- a short paragraph: why this matters, in plain sentences -->

## Notes
<!-- the main content as prose: what was done, why, on what data. Never
     results or metrics. Normal sentences, important thing first. A few
     short paragraphs beat a wall of bullets. -->

## Links
- [[related_card_1]]
- [[related_card_2]]
```

### 4b. The vault is a lab notebook, not a scoreboard

A card records what was done in a session — never what the data showed. Do
not write results into cards: no metric values, no error statistics, no
timings, no comparisons between methods ("5% better than the esoteric
baseline"), and no qualitative stand-ins for numbers ("beat X handily",
"the fits look great"). Cards that contain these rot immediately — every
number is stale by the next run — and they turn the vault into a leaderboard.

A card carries exactly this, and nothing else:

- which experiment ran and what it does,
- why it was run,
- which job ID ran it and how it ended (completed / failed / requeued),
- which dataset or recording it touched,
- any decision made about what to do next.

Results, numbers, and conclusions live in `{project}/runs/`, `{project}/out/`,
the figures, and the eventual paper. The card is an index to where the work
happened — point at those artifacts, don't restate them. The vault has already
been manually scrubbed of metric-laden cards once (see `MEMORY/` git history,
commit `d93df07`); don't make that necessary again.

### 5. Register it in the project hub

Append one line to `/scratch/ap7151/MEMORY/<project>/MEMORY.md`:

```markdown
- [Short Name](<file>.md) — what it is, one clause
```

The hub is a table of contents, not content. Keep the entry to one line — one
clause saying what the card is, where its code or figures live, and its status
(complete / active / superseded). Job IDs are welcome when they are the
load-bearing pointer; results, metric values, and changelog paragraphs are
not — the card holds the prose. The link must resolve to a file that exists;
never write a hub line for a card you did not create in the same pass.
Collaborator cards (`user_<handle>.md` about another person) keep their fixed
entry format:

```markdown
- [user_<handle>](user_<handle>.md) []: Memory card link for the experiments run by <name>
```

Sort new lines near the top; group with `##` headers (Active / Reference / Lessons) if
the hub is growing. Mark priority in the summary: `## HIGH PRIORITY — ...`. The central
hub `/scratch/ap7151/MEMORY/MEMORY.md` already links the project subfolder — no central
edit needed for ordinary cards.

### 6. Link related cards both ways

Add `[[...]]` or relative `[...](...)` links in the new card and in each related card
so they are reachable from both directions.

## NEVER

- Create a second card for a fact that already exists — update the first one.
- Write a card with an unclear title (you should be able to summarize it in one line).
- Skip the `MEMORY.md` line (an unregistered card is lost forever).
- Duplicate content between two cards — store the fact once and link everywhere else.
- Write results into a card: no metrics, no error statistics, no timings, no
  method comparisons, not even qualitative ones ("beat X", "looks great").
  A card says what ran, why, on what data, with which job ID — that is it.
- Let a hub entry grow beyond one clause, or link to a file that does not
  exist — the hub is a table of contents, and a phantom link is a lost card.
