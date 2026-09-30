# Memories

Persistent operational state for the RunPod Manager. Read at the start of every
session; update whenever something changes. Conventions and what's worth
recording: `SYSTEM_PROMPT.md` §11.

## Files

- _(none yet — add one line per file as they're created)_

## Conventions

- **Never store secrets.** Record where a secret lives, not its value.
- **Dates are absolute** (`2026-09-30`), never "yesterday" / "last week".
- Terse factual bullets; one fact per bullet. Delete what stops being true.
- When reality contradicts a memory, fix the memory and note what changed.

## Log of changes

- 2026-09-30 — Repo initialized as the runpodctl-based RunPod Manager
  (`SYSTEM_PROMPT.md`, `bin/rp`, `.env.example`). No account state recorded yet.
