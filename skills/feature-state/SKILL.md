---
name: feature-state
description: Use on demand to distill the current state of a feature into a dated snapshot file. Consolidates multiple design specs and implementation history into a single reference file that future brainstorming sessions can point to instead of re-reading every spec.
---

# Feature State (craft)

Produce a dated, semantic snapshot of a feature. The snapshot is a compact, high-signal anchor that a future session loads instead of rereading every spec.

**Default artifact root:** `docs/craft/`. A project's `CLAUDE.md` (or equivalent) may override this path.

**Announce at start:** "I'm using the craft:feature-state skill to capture the current state of <feature>."

## When to use

- End of an implementation session, when meaningful state has changed.
- End of a brainstorming session, when new canonical decisions emerged.
- On demand when the spec pile is noisy and a distilled reference would help.

This skill is **on-demand**. Other skills may prompt to invoke it but must not call it automatically.

## Process

### 1. Gather inputs in a single batched message

Before reading any file, do a lightweight search for files referencing the slug (if the user named one) or the domain they described. Then send **one** message that packages the slug, scope, file list, and anything else you need in one round-trip. The aim is one reply from the user, not three.

Why this matters: human attention is the bottleneck. Three sequential yes/no gates cost three context switches for the user and don't yield more information than a single structured ask. Resist the urge to ask step-by-step just because each question is individually easy.

Template for the first message:

```
I'm going to distill the state of <slug> into a dated snapshot. A few inputs to confirm, then I'll proceed:

1. Feature slug: <slug>  ← confirm or correct
2. Scope: <update mode because features/<slug>/YYYY-MM-DD-<slug>-state.md exists>
   OR <first-time distillation because no prior state file found>
   ← override if you want a full re-distillation either way
3. Files I found referencing this feature: <compact list>
   ← anything missing I should include?
4. Specs/task logs in scope: <list from <root>/specs/ and <root>/tasks/>
   ← if more than 3-4, flag which are actually relevant

Reply with corrections or "go ahead".
```

**Auto-detect scope:** if `<root>/features/<slug>/YYYY-MM-DD-<slug>-state.md` exists, default to **update mode** — diff what changed since that file (new specs, new task logs, code drift) and revise only the affected sections. Otherwise default to **first-time** distillation. State your choice in the message above; the user can override.

Wait for the reply before reading heavy content. A state file with invented details is worse than one with a flagged open question.

### 2. Read code first, then specs and task logs

Code is the source of truth — specs describe intent, code describes reality. Read confirmed implementation files first, then:
- `<root>/specs/` — design specs from `craft:brainstorm`.
- `<root>/tasks/` — dated implementation-session logs from `craft:implement`. Task logs often hold rationale that lives nowhere else; do not skip them.
- `<root>/features/<slug>/` — most recent state file and `overview.md`.

When you hit an unclear point, **ask**. If code and spec disagree and the reason isn't obvious from git log or task logs, ask which is current. If feature status (WIP / shipping / stable) is ambiguous, ask. Do not guess.

### 3. Distill, don't summarize

Distillation re-states only what still matters today, using current code as the source of truth.
- If a spec said "we will do X" and code now does Y, document Y. Note divergence only if intentional.
- If a spec raised an open question that has since been answered, record the answer, not the question.
- Architecture description: 2-4 paragraphs, no code dumps.

### 4. Write the new state file

Create the directory with `mkdir -p <root>/features/<slug>/` if needed, then write to `<root>/features/<slug>/YYYY-MM-DD-<slug>-state.md`.

Use the template in `templates/state-file.md`. Set `Supersedes:` to the previous state file's filename (not full path), or "first state file" if none.

**Keep all dated state files for history.** The newest is authoritative; older ones are an audit trail.

### 5. Offer to update `overview.md`

Ask:

> Any product, architectural-invariant, or external-constraint decisions worth logging in `features/<slug>/overview.md`?

If yes, read `references/overview-md.md` for the filter rules, bucket definitions, and template, then append. Do not mass-rewrite; only append new entries.

### 6. Hand off

Report:
- Path to the new state file.
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
| "I'll delete the old state file" | No. Keep all for history. |

## Integration

- Paired with `craft:implement`, which prompts (does not auto-invoke) this skill at finish.
- Output feeds brainstorming sessions: load the files listed in "Load This In Next Session" instead of the full spec pile.
