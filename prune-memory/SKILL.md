---
name: prune-memory
disable-model-invocation: false
description: Archive stale or orphaned memory cards. Use when the user asks to clean up the memory vault, or when MEMORY.md has grown unwieldy. Finds unlinked cards, flags duplicates, and archives with confirmation.
---

# prune-memory

Tidy the memory vault without losing anything. There is no real deletion system —
forgetting happens by unlinking and archiving.

## Steps

### 1. Locate the vault

The vault is central: `/scratch/ap7151/MEMORY/`, with a systemwide hub (`MEMORY.md`)
linking per-project subfolders (`slt/`, `macro-micro/`, `ssl-codebook/`, ...), each
with its own `MEMORY.md`. Determine the current project from the cwd (match loosely —
e.g. `MICRO_TO_MACRO` vs `macro-micro` — or ask the user), and prune that subfolder.
Prune other subfolders only when the user asks explicitly.

### 2. Find orphans — cards not linked from the project hub

```
for f in /scratch/ap7151/MEMORY/<project>/*.md; do
  [ "$(basename $f)" = "MEMORY.md" ] && continue
  grep -q "$(basename $f)" /scratch/ap7151/MEMORY/<project>/MEMORY.md || echo "ORPHAN: $f"
done
```

### 3. Flag duplicate IDs / near-duplicate cards

```
ls /scratch/ap7151/MEMORY/<project>/ | sed 's/-[0-9-]*$//' | sort | uniq -d
```

Also list any `session-<NNN>-*` files sharing the same number.

### 4. Present findings and ask

Show the orphans, duplicates, and the current hub length. Also flag any card
that reads like a scoreboard — dominated by metric values, error statistics,
timings, or method comparisons ("5% better than X") rather than what was
done. Those violate the vault's rule (lab notebook, not scoreboard; see
AGENTS.md Memory) and should be offered a scrub — strip the results, keep the
job IDs, what ran, why, and the datasets touched — before archiving is
considered. For each finding, ask the user: scrub, archive, or delete?

- **Scrub** (for metric-laden cards): rewrite the card in place, keeping job
  IDs, what the experiment does, why it was run, the dataset, and decisions —
  dropping every result value and comparison.
- **Archive** (default): move to `/scratch/ap7151/MEMORY/<project>/archive/`, drop any hub line.
- **Delete**: only with explicit, per-file confirmation.

### 5. Rebalance the hub

If the project's `MEMORY.md` exceeds ~1 page of entries: regroup with `##` headers, move old rows to
an `## Archived` section, and prune a light quarterly tidy — never more.

While in there, repair the two hub rot patterns:

- **Bloated entries.** A hub line is one clause (`- [Name](file.md) — what it
  is`). If an entry has grown into a paragraph, a dated changelog, or wrapped
  onto extra lines, collapse it back to one clause — the card holds the prose.
- **Phantom links.** Check every hub link resolves (`test -f`). A link to a
  card that never existed gets dropped; if the lesson text survives only in
  the old hub line, it is recoverable from the vault's git history.

Before a bulk hub rewrite, commit a snapshot of the vault first, so the
history always shows snapshot → rewrite.

## NEVER

- Auto-delete anything. Archiving only, and only after the user confirms.
- Forget to update `MEMORY.md` when a card leaves the active map.