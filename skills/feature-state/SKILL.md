---
name: feature-state
description: Use on demand to distill the current state of a feature into a dated snapshot file. Consolidates multiple design specs and implementation history into a single reference file that future brainstorming sessions can point to instead of re-reading every spec.
---

# Feature State (craft)

Produce a dated, semantic snapshot of a feature's current state. The snapshot is a compact, high-signal anchor that a future session can load instead of rereading every spec.

**Default artifact root:** `docs/craft/`. A project's `CLAUDE.md` (or equivalent project instructions file) may override this path. If overridden, use the override everywhere.

**Announce at start:** "I'm using the craft:feature-state skill to capture the current state of <feature>."

**Asking questions:** Ask when distillation would require guessing - for example, when code and spec disagree and the reason isn't in git history, when a status is ambiguous, or when an open question's resolution can't be found in code or task logs. Otherwise, record what you see.

## When to use

- End of an implementation session, when meaningful state has changed.
- End of a brainstorming session, when new canonical decisions emerged and the next session needs a cleaner starting point.
- On demand when the spec pile is getting noisy and a distilled reference would help.

This skill is **on-demand**. Other skills (like `craft:implement`) may prompt to invoke it but must not call it automatically.

## Pre-flight

Create `docs/craft/features/<slug>/` with `mkdir -p` when you reach step 5 (writing the state file). No y/n prompt; no `.gitignore` ceremony - those live in the skills that actually produce ignorable artifacts (`craft:brainstorm` for `.craft/` mockups, `craft:implement` for `.worktrees/`).

## Step 0 — Ask before touching anything

Do not read any file until you have answers to these. Do not skip.

Ask the user:
1. **Feature slug** (human-chosen, e.g. `assistant-v2`). This becomes the directory name under `docs/craft/features/`.
2. **Scope** — determined automatically by whether a previous state file exists under `docs/craft/features/<slug>/`:
   - **No previous state file** → full distillation. Create a fresh state file from scratch.
   - **Previous state file exists** → update mode. Diff what has changed since that file (new specs, new task logs, code drift) and revise only the affected sections. Keeps the state file short and avoids re-deriving everything every session.
   - Confirm the choice with the user in one line before proceeding; they can override (e.g. "do a full re-distillation anyway" if the previous state is stale).
3. **"I'm going to search the codebase for relevant files first — any areas you want to make sure I include?"**

## Step 1 — Discover files, show, confirm

Search the codebase for files related to the feature slug and its domain. Cast a wide net - include all layers relevant to this project:

- Implementation files (source modules, libraries, or whatever the project's primary code lives in).
- Tests covering those files.
- Configuration, schema, or data files tied to the feature.
- Entry points, interfaces, or boundaries that expose or consume the feature.

Adapt to the project's actual layout. A library project may be mostly implementation + tests; a service may have layered code and schemas; a data/research project may be notebooks, scripts, and datasets. Use what's there, not a fixed template.

**Display the found file list to the user.** Then ask:

> "These are the files I found. Are there others I should include? Is there anything missing or any context I should know about before I start reading?"

Wait for confirmation before proceeding. The user is the authority on completeness — do not assume the search result is exhaustive.

## Step 2 — Find specs and task logs

Search for design specs AND implementation-session task logs related to the feature. Check the project's `CLAUDE.md` for the artifact root — it may not be `docs/craft/` if the project has overridden it.

Look in:
- `docs/craft/specs/` - design specs produced by `craft:brainstorm`.
- `docs/craft/tasks/` - dated implementation-session logs produced by `craft:implement`. Task logs are often the only place a small decision's rationale lives, so do not skip them.

Display the found specs and task logs to the user. If more than 3-4 of either are found, ask which are in scope before reading them all.

Also check `docs/craft/features/<slug>/` for:
- The most recent `YYYY-MM-DD-<slug>-state.md`, if any
- `overview.md`, if it exists

## Step 3 — Read: code first, then specs and task logs

Read the confirmed implementation files first. Code is the source of truth — specs describe intent, code describes reality.

Then read the relevant specs and task logs for context on decisions and rationale that are not visible from the code alone (e.g. why A was chosen over B, what was explicitly deferred, what a task session cut from scope).

**When you hit an unclear point, ask.** If code and spec disagree and the reason isn't obvious from git log or task logs, ask the user which is current before recording either. If the feature's status (WIP / shipping / stable) is ambiguous, ask. Do not guess — a state file with invented details is worse than a state file with a flagged open question.

## Step 4 — Distill, do not summarize

Summaries copy the spec text. Distillation re-states only what still matters today, using current code as the source of truth. Rules:
- If a spec says "we will do X" and the code now does Y, document Y. Note the divergence only if it was intentional.
- If a spec raised an open question that has since been answered (in a later spec or in code), record the answer, not the question.
- Keep architecture description to 2-4 paragraphs. No code dumps.

## Step 5 — Write the new state file

Path: `docs/craft/features/<slug>/YYYY-MM-DD-<slug>-state.md`

Template:

```markdown
# <Feature Name> - State (YYYY-MM-DD)

**Status:** WIP | shipping | stable
**Owner:** <name>
**Supersedes:** YYYY-MM-DD-<slug>-state.md (or "first state file")

## Goal
One paragraph on what this feature is and why it exists.

## Current Architecture
2-4 paragraphs. Describe the shape: main components, how they collaborate, critical boundaries. Not a code dump. Pull from code, not from old specs.

## Key Files
- path/to/module.<ext> - responsibility
- path/to/other.<ext> - responsibility

## Decisions Made (and rationale)
- Chose A over B because ...
- Deferred C because ...

## Known Gaps / Open Questions
- Not yet built: ...
- Stubbed: ...
- Next session should tackle: ...

## Load This In Next Session
To get full context without re-reading specs, load:
- This file
- features/<slug>/overview.md (if it exists)
- 2-3 most critical implementation files - the ones a new session would need open to make good decisions.

Omit this section entirely on a first state file if there are no meaningful implementation files yet (design-only feature). Do not leave the placeholder bullet in the output.

## Related Specs
- specs/YYYY-MM-DD-<topic>-design.md - still relevant: <which parts>
- specs/YYYY-MM-DD-<topic>-design.md - superseded
```

Set `Supersedes:` to the filename of the previous state file (not a full path).

## Step 6 — Do not delete old state files

Keep all dated state files for history. The newest one is authoritative. Older ones stay as an audit trail.

## Step 7 — Offer to update `overview.md`

After writing the state file, ask:

> Any product, architectural-invariant, or external-constraint decisions worth logging in `features/<slug>/overview.md`?

`overview.md` is different from a state file. It is a small, stable list of decisions that outlive any given implementation. It is not dated - it evolves in place.

**The filter (one line):** if a future reader could answer this by reading the code, it does NOT belong in `overview.md`.

**Log an entry only if it fits one of these three buckets:**

1. **Product / business** - why the feature exists, who it serves, what counts as done, explicit scope boundaries. Recoverable from stakeholder conversations, not from code.
   - Example: "This feature exists to satisfy the EU battery passport requirement - not as a generic audit log."
2. **Architectural invariants** - boundaries, ownership, cross-cutting patterns that constrain future decisions.
   - Example: "Module A must never call Module B directly - all traffic goes through the event bus."
   - Example: "All new features in this domain must gate behind a feature-flag entry."
3. **External constraints** - compliance, legal, SLAs, contracts with other teams or systems.
   - Example: "Response time must stay under 200ms; agreed with the mobile team for their cache TTL."

**Explicitly excluded (redirect to the right place):**

- Function names, file paths, signatures, type shapes → read the code.
- Naming or refactor choices ("renamed X to Y") → git history.
- Implementation mechanics ("uses a dict keyed by user_id") → code.
- Session-level "we deferred X" → that's what the dated state file's *Known Gaps* section is for.
- Rejected alternatives, unless the rationale is load-bearing for future decisions ("we considered X, rejected because Y" where Y will come up again).

If `overview.md` does not exist yet and the user wants to add entries, create it with this shape:

```markdown
# <Feature Name> - Overview

Long-lived decisions for <feature>. Short, stable, evolves in place. Every entry traces back to a spec or a session, and fits one of: product/business, architectural invariant, or external constraint.

## Product / Business
- <Decision>. Source: specs/YYYY-MM-DD-<topic>-design.md

## Architectural Invariants
- <Invariant>. Source: specs/YYYY-MM-DD-<topic>-design.md

## External Constraints
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md or external doc.

## Open Questions
- <Question>.

## History
- YYYY-MM-DD: added <decision> under <bucket> (source: spec or session).
```

When appending to an existing `overview.md`, put each new entry under the correct bucket and add a `History` line. Do NOT rewrite existing entries unless the user explicitly asks.

## Step 8 — Hand off

Report to the user:
- Path to the new state file.
- Whether overview was updated.
- Remind: in the next brainstorming session, load the files listed under "Load This In Next Session" — not the full spec pile.

## Red Flags

| Thought | Reality |
|---------|---------|
| "I know which files matter, I'll read them directly" | No. Search first, show the list, ask the user to confirm before reading. |
| "The inputs aren't critical, I'll start reading" | No. Ask for slug, scope, and areas to cover before touching any file. |
| "I found the specs, I'll read them all" | Show the list first. Ask which are in scope if more than 3-4. |
| "I will just copy the latest spec and call it state" | No. Distill from code + specs, do not summarize a single doc. |
| "I will update overview.md without asking" | No. Overview is stable. Only append on explicit user confirmation. |
| "I'll log the class structure / function signatures for future reference" | No. That's code. Overview is for product, architectural-invariant, or external-constraint decisions only. |
| "I will delete the old state file" | No. Keep all for history. |
| "I will write this without reading code" | No. Specs drift. Code is truth. Read code before specs. |
| "The spec had 5 open questions, I will list them all" | Only list ones still open today. Answer the rest from current code. |

## Integration

- Paired with `craft:implement`, which prompts (does not auto-invoke) this skill at finish.
- Feeds brainstorming sessions: the user should start by loading the files listed in "Load This In Next Session" instead of the full spec pile.
