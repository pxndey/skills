---
name: find-memory
disable-model-invocation: false
description: Find and read memory cards in the central memory vault. Use when the user says "read memories on ...", "what do we know about ...", "do we have a note on ...", or "resume <topic>" to look something up. Greps the vault and summarizes what's recorded.
---

# find-memory

Search the central linked-Markdown memory vault and summarize what is recorded.

The vault is central: `/scratch/ap7151/MEMORY/`, with a systemwide hub (`MEMORY.md`)
linking per-project subfolders (`slt/`, `macro-micro/`, `ssl-codebook/`, ...), each
with its own `MEMORY.md`.

## Steps

### 1. Locate the vault

The vault is always `/scratch/ap7151/MEMORY/`. Determine the current project from the
cwd: list `ls /scratch/ap7151/MEMORY/` and match the cwd (project directories live at
`/scratch/ap7151/<PROJECT>/`) against the subfolder names — the names may differ slightly
(e.g. `MICRO_TO_MACRO` vs `macro-micro`), so match loosely or ask the user. If the topic
spans projects, search the whole vault.

### 2. Search card contents

```
grep -ri "<topic>" /scratch/ap7151/MEMORY/ --include="*.md" -l
```

### 3. Cross-check the hub

```
grep -i "<topic>" /scratch/ap7151/MEMORY/<project>/MEMORY.md
```

Also grep the central hub `/scratch/ap7151/MEMORY/MEMORY.md` — its one-line summaries
tell you which card to open without opening every file.

### 4. Present

- Show each hit's summary line from the hub `MEMORY.md` (one line each — don't dump whole files).
- Read the 1-2 most relevant cards in full and present:
  - What's known
  - What's uncertain / outdated
  - Which cards link to/from it (the neighborhood)
- If results look stale (dates older than the current work), flag for an update.
- Cards record what was done, not what the data showed — they carry job IDs,
  datasets, and decisions, not metrics or method comparisons. If you need a
  number, look in `{project}/runs/`, `{project}/out/`, or the figures; don't
  expect it from the vault, and don't add it there.

### 5. If nothing found

Say so explicitly and offer to create a `new-memory` card in the current project's subfolder.

## Tips

- Filenames are the primary retrieval key: `project_xyz.md` = topic xyz.
- Find by type with a prefix glob: `ls /scratch/ap7151/MEMORY/<project>/project_*.md` or
  `.../session-*.md`.
- Use `grep -ri "<topic>" /scratch/ap7151/MEMORY/*/MEMORY.md` as a quick "which card is it"
  lookup across all project hubs.
- Hub entries are one clause per card by design. If a line has grown into a
  paragraph or its link points at a missing file, flag it for a `prune-memory`
  pass instead of adding more text to it.