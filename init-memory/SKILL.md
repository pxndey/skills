---
name: init-memory
disable-model-invocation: false
description: Register a new project in the central memory vault. Use when the user says "init memory", "set up memory", "start a memory vault", or starts work on a new project that has no memory subfolder yet. Creates a new subfolder inside MEMORY/ with seed cards and registers it in the central MEMORY.md.
---

# init-memory

Register a new project in the central linked-Markdown memory vault. Plain Markdown +
a folder convention — zero infrastructure.

The vault is central: `/scratch/ap7151/MEMORY/`, with a systemwide hub (`MEMORY.md`)
linking per-project subfolders (`slt/`, `macro-micro/`, `ssl-codebook/`, ...), each
with its own `MEMORY.md`.

## What it creates

- `/scratch/ap7151/MEMORY/<project>/` — the new project's memory subfolder
- `.../MEMORY.md` — the project hub: one `[Name](file.md) — what it is, one clause` per card, every link resolving to a file created in the same pass
- `.../project_overview.md` — seed card
- `.../user_me.md` — seed card (copy from an existing project and adjust)
- `.../archive/` — for future pruning (cards never get auto-deleted)
- a `- [Name](<project>/MEMORY.md) — ...` line in the central `/scratch/ap7151/MEMORY/MEMORY.md`

## Writing style for card content

Cards are for a person, not a database. Use plain, natural, relaxed language — write
like someone explaining what happened to another person.

- Do not write in terse engineering-status-report or incident-report style. Avoid
  dense, compressed technical prose like "failure → root cause → fix → requeued"
  unless the user explicitly asks for a postmortem or formal technical report.
- Avoid labels like "The failure:", "Root cause:", "The fix:", "Requeued:",
  "Current state:" by default. Just tell what happened naturally.
- Don't compress several technical facts into one long sentence. Explain the
  important thing first, then add details only when they help.
- Technical terms are fine when they're actually useful; skip jargon, internal
  shorthand, and implementation terminology that isn't needed.
- Headings are still fine for organizing a card, but the text under each heading
  should be normal sentences.

**The vault is a lab notebook, not a scoreboard.** Cards record what was done
in a session — never what the data showed. No metric values, no error
statistics, no timings, no comparisons between methods ("5% better than the
esoteric baseline"), and no qualitative stand-ins for numbers ("beat X
handily", "the fits look great"). A card carries: which experiment ran and
what it does, why it was run, which job ID ran it and how it ended, which
dataset it touched, and any decision about what to do next — that is all.
Results live in `{project}/runs/`, `{project}/out/`, the figures, and the
eventual paper; the card is an index to that work, not a restatement of it.

## Steps

### 1. Pick the project name

- Use the lowercase hyphenated name of the current project directory if the cwd is
  inside one (e.g. `MICRO_TO_MACRO` → `macro-micro`), otherwise ask the user.
- **Guard against duplicates**: list `ls /scratch/ap7151/MEMORY/` first. If a subfolder
  already exists for this project (match loosely), STOP and report its path — do not
  create a second one.

### 2. Create the skeleton

Create `/scratch/ap7151/MEMORY/<project>/` and `.../archive/`. Write files with the
Write tool (never shell heredocs).

`/scratch/ap7151/MEMORY/<project>/MEMORY.md`:

```markdown
# MEMORY.md
<project name>

## Project Map
- [Project Overview](project_overview.md) — what this project does, one line
- [User Profile](user_me.md) — working style, communication preferences, must-haves

## Sessions
(one `- [Name](session-NNN-slug.md) — summary` per active thread)
```

`/scratch/ap7151/MEMORY/<project>/project_overview.md`:

```markdown
# Project Overview

## Goal

<!-- one or two plain sentences: what this project is trying to do and why -->

## How it's put together

<!-- a short paragraph describing the main pieces and how they fit together.
     Write it as prose, not a table or bullet dump. -->

## Where things stand

<!-- a short paragraph: what works, what doesn't, what's next.
     Write it like you're explaining it to a person, not filing a report. -->
```

`/scratch/ap7151/MEMORY/<project>/user_me.md`:

```markdown
# User Profile

## Working style

<!-- a short paragraph, e.g. neuroscience researcher, works in Python,
     prefers simple approaches over clever ones -->

## Communication preferences

<!-- a short paragraph describing how they like updates:
     plain natural sentences, no jargon, important thing first -->

## Must-haves

<!-- a short paragraph: the things that always need to be true,
     e.g. every number comes from real data, verify with overlays -->
```

### 3. Register the project in the central hub

Append one line to the Project Map section of `/scratch/ap7151/MEMORY/MEMORY.md`:

```markdown
- [<Project Name>](<project>/MEMORY.md) — one-line summary of what the project is
```

### 4. Confirm

Report the subfolder path, the files created, and the three-card workflow: ADD
(new-memory), QUERY (find-memory), RESUME (resume-session). Offer to seed the first
card by asking for the project's one-line goal.
