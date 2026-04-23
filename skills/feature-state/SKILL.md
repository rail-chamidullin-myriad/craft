---
name: feature-state
description: Distill the current state of a feature into a dated snapshot so future sessions can load one anchor file instead of re-reading every historical spec. Use this whenever the user says "snapshot this feature", "update feature state", "capture what we have", "distill the state", "the spec pile is getting noisy", "re-baseline the feature", at the end of an implementation session with meaningful state changes, at the end of a brainstorming session with new canonical decisions, or whenever a compact high-signal anchor for a feature would save re-reading the spec pile next time. On-demand only — other skills may prompt to invoke it but must not call it automatically. Consolidates multiple design specs and implementation history into a single dated reference file and can append canonical decisions to features/<slug>/overview.md.
---

# Feature State (craft)

Produce a dated, semantic snapshot of a feature. The snapshot is a compact, high-signal anchor that a future session loads instead of rereading every spec.

**Memory:** this skill reads specs and prior state from craft memory and writes new snapshots there. Path, layout, and invariants are defined in [`../../references/memory.md`](../../references/memory.md). File roles and read/write protocol for `overview.md` and `snapshots/*.md` are defined in [`../../references/feature-files.md`](../../references/feature-files.md). Read both before any memory access. `<memory-root>` refers to the path defined in `memory.md` (also appears as `<root>` in path templates below — same directory).

**Announce at start:** "I'm using the craft:feature-state skill to capture the current state of <feature>."

## Process

### 1. Gather inputs in a single batched message

Before reading any file, do a lightweight search for files referencing the slug (if the user named one) or the domain they described. Then send **one** message that packages the slug, scope, file list, and anything else you need in one round-trip. The aim is one reply from the user, not three.

Why this matters: human attention is the bottleneck. Three sequential yes/no gates cost three context switches for the user and don't yield more information than a single structured ask. Resist the urge to ask step-by-step just because each question is individually easy.

Template for the first message:

```
I'm going to distill the state of <slug> into a dated snapshot. A few inputs to confirm, then I'll proceed:

1. Feature slug: <slug>  ← confirm or correct
2. Scope: <update mode because features/<slug>/snapshots/ has entries>
   OR <first-time distillation because no prior snapshot found>
   ← override if you want a full re-distillation either way
3. Files I found referencing this feature: <compact list>
   ← anything missing I should include?
4. Specs in scope: <list from <root>/specs/>
   ← if more than 3-4, flag which are actually relevant

Reply with corrections or "go ahead".
```

**Auto-detect scope:** if `<root>/features/<slug>/snapshots/` contains any files, default to **update mode** — diff what changed since the newest snapshot (new specs, new commits since its header date, code drift) and revise only the affected sections. Otherwise default to **first-time** distillation. State your choice in the message above; the user can override.

Wait for the reply before reading heavy content. A snapshot with invented details is worse than one with a flagged open question.

### 2. Read code first, then specs

Code is the source of truth — specs describe intent, code describes reality. Read confirmed implementation files first, then:
- `<root>/specs/` — design specs from `craft:brainstorm`.
- `<root>/features/<slug>/overview.md` (if it exists) plus the newest `<root>/features/<slug>/snapshots/*.md` (if any). See the read protocol in `references/feature-files.md`.
- **Branch / merge state — canonical three-command probe.** Run these before diving into commit bodies so you know what is shipped vs. WIP:
  - `git rev-parse --abbrev-ref HEAD` — current branch.
  - `git log --oneline <main>..HEAD` — commits ahead of main (WIP / unmerged).
  - `git log --since="<last-snapshot-date>" --oneline <main> -- <feature-paths>` — commits that landed on main since the previous snapshot.
  Record the three commands and their output in the snapshot's `Branch / Merge Status` section (see the template). That section is how a future reader tells what shipped vs. what is still on a branch.
- If `gh pr list` / `gh pr view` is available, add PR numbers and states. Don't block on it — TLS or auth failures are common; fall back to commit messages (PR numbers usually appear in the subject line).

When you hit an unclear point, **ask**. If code and spec disagree and the reason isn't obvious from git log, ask which is current. If feature status (WIP / shipping / stable) is ambiguous, ask. Do not guess.

### 3. Distill, don't summarize

Distillation re-states only what still matters today, using current code as the source of truth.
- If a spec said "we will do X" and code now does Y, document Y. Note divergence only if intentional.
- If a spec raised an open question that has since been answered, record the answer, not the question.
- Architecture description: 2-4 paragraphs, no code dumps.

### 4. Write the new snapshot

Follow the write protocol in `references/feature-files.md`: `mkdir -p <root>/features/<slug>/snapshots/`, then write `<root>/features/<slug>/snapshots/YYYY-MM-DD.md` using today's date and the template in [`../../references/templates/snapshot.md`](../../references/templates/snapshot.md). Set `Supersedes:` to the previous snapshot's filename (e.g. `snapshots/2026-04-19.md`), or `"first snapshot"` if none exists.

Never edit or delete an existing snapshot. If a snapshot already exists for today, append a suffix (e.g. `2026-04-22b.md`).

### 5. Offer to update `overview.md`

Ask:

> Any product, architectural-invariant, or external-constraint decisions worth logging in `features/<slug>/overview.md`?

If yes, read [`../../references/overview.md`](../../references/overview.md) for the filter rules, bucket definitions, and template, then append. Do not mass-rewrite; only append new entries.

### 6. Hand off

Report:
- Path to the new snapshot.
- Whether `overview.md` was updated.
- Reminder: in the next brainstorming session, load the files listed under "Load This In Next Session" — not the full spec pile.

## Red Flags

| Thought | Reality |
|---------|---------|
| "I'll ask slug first, then file list, then scope — one at a time" | No. One batched message. Three serial asks is the failure mode this skill exists to prevent. |
| "I know which files matter, I'll start reading" | No. Search and show the list in the first message. |
| "I'll read every spec I find" | If more than 3-4, flag which are actually relevant in the first message. |
| "I'll just copy the latest spec" | No. Distill from code + specs. Code is truth. |
| "I'll write this without reading code" | No. Specs drift. Read code first. |
| "I'll list all the spec's old open questions" | Only ones still open today. Answer the rest from current code. |
| "I'll update overview.md without asking" | No. Overview is stable. Only append on explicit yes. |
| "I'll log function signatures in overview.md" | No. That's code. Overview is product/invariant/constraint only. |
| "I'll delete old snapshots" | No. `snapshots/*.md` is the audit trail. |

## Integration

- Paired with `craft:implement`, which prompts (does not auto-invoke) this skill at finish.
- Output feeds brainstorming sessions: load the files listed in "Load This In Next Session" instead of the full spec pile.
