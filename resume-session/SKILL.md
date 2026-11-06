---
name: resume-session
disable-model-invocation: false
description: Resume a previous working session — load session notes from the memory vault, check SLURM job status, and show what's pending. Use when the user says "resume session" or wants to pick up where they left off.
---

# resume-session

Resume a previous working session from the current project's memory subfolder in the
central vault.

The vault is central: `/scratch/ap7151/MEMORY/`, with a systemwide hub (`MEMORY.md`)
linking per-project subfolders (`slt/`, `macro-micro/`, `ssl-codebook/`, ...), each
with its own `MEMORY.md`.

## Steps

### 1. Locate the vault and the session file

The vault is always `/scratch/ap7151/MEMORY/`. Determine the current project from the
cwd: list `ls /scratch/ap7151/MEMORY/` and match the cwd (project directories live at
`/scratch/ap7151/<PROJECT>/`) against the subfolder names — the names may differ slightly
(e.g. `MICRO_TO_MACRO` vs `macro-micro`), so match loosely or ask the user. Sessions are
`session-<NNN>-<slug>.md` files in `/scratch/ap7151/MEMORY/<project>/`.

- If `$ARGUMENTS` is a number (e.g. `010`), match `MEMORY/<project>/session-010-*.md`.
- If it's a name (e.g. `atlas`), glob `MEMORY/<project>/session-*.md` and grep contents for the name.
- If no argument, list all available sessions (`ls /scratch/ap7151/MEMORY/<project>/session-*.md`).

### 2. Load session context

Read the session file and summarize:

- **Session name and status** (Active / Complete)
- **What was accomplished** (completed items)
- **What's pending** (unchecked `[ ]` items in Next Steps)
- **Key files created / modified**

### 3. Check job status

Extract every job ID mentioned in the session file, then:

- Queue: `squeue -u $USER`
- Completed/failed: `sacct -j <id> --format=JobID,JobName%30,State,Elapsed,ExitCode -n`
- For recently completed jobs, show the last ~20 lines of their log files.

Use one-shot commands only — never `watch` or tight poll loops on HPC.

### 5. Present the resume summary

Report each job's state (running / completed / failed / requeued) and its
outcome — not its numbers. Do not extract metrics (val_loss, accuracy, error
statistics, timings) from logs, and do not read results into the summary: the
vault is a lab notebook, not a scoreboard. If the user wants to know how a
run performed, show the figures and logs from `{project}/out/` and
`{project}/runs/` instead.

```
Session 010: <title> — Status: Active (last updated YYYY-MM-DD)
Jobs since last update:
  2733255 (name): RUNNING — 8h in
  2743637 (name): FAILED — all trials collapsed, logs show <what the failure was>
Pending tasks:
  [ ] ...
Ready to continue. What would you like to work on?
```

### 6. Update and record

Update the session card with what happened since the last entry — appended,
dated, never overwritten: which jobs ran or finished (job IDs and their
outcome state), what was done about failures, and any decisions made. Do not
add metric values or result summaries to the card. Refresh the session line
in `/scratch/ap7151/MEMORY/<project>/MEMORY.md` if its one-line summary
changed — keeping it one clause, never a changelog.