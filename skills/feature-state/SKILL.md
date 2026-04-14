---
name: feature-state
description: Use on demand to distill the current state of a feature into a dated snapshot file. Consolidates multiple design specs and implementation history into a single reference file that future brainstorming sessions can point to instead of re-reading every spec.
---

# Feature State (craft)

Produce a dated, semantic snapshot of a feature's current state. The snapshot is a compact, high-signal anchor that a future session can load instead of rereading every spec.

**Default artifact root:** `docs/craft/`. A project's `CLAUDE.md` or `CLAUDE.local.md` may override this path. If overridden, use the override everywhere.

**Announce at start:** "I'm using the craft:feature-state skill to capture the current state of <feature>."

## When to use

- End of an implementation session, when meaningful state has changed.
- End of a brainstorming session, when new canonical decisions emerged and the next session needs a cleaner starting point.
- On demand when the spec pile is getting noisy and a distilled reference would help.

This skill is **on-demand**. Other skills (like `craft:implement`) may prompt to invoke it but must not call it automatically.

## Step 0 — Ask before touching anything

Do not read any file until you have answers to these. Do not skip.

Ask the user:
1. **Feature slug** (human-chosen, e.g. `assistant-v2`). This becomes the directory name under `docs/craft/features/`.
2. **Scope** — full re-distillation, or update to the last state file? Full is safer if more than one spec or implementation session has happened since the last state.
3. **"I'm going to search the codebase for relevant files first — any areas you want to make sure I include?"**

## Step 1 — Discover files, show, confirm

Search the codebase for files related to the feature slug and its domain. Look across:
- Backend: service modules, DB models, repositories, API routers, predictors, utilities
- Frontend: feature components, page files, hooks, REST methods
- Tests: test files for the above
- Config/migrations: any schema or config files tied to the feature

**Display the found file list to the user.** Then ask:

> "These are the files I found. Are there others I should include? Is there anything missing or any context I should know about before I start reading?"

Wait for confirmation before proceeding. The user is the authority on completeness — do not assume the search result is exhaustive.

## Step 2 — Find specs

Search for design specs and implementation plans related to the feature. Check `CLAUDE.md` for the project's spec location — it may not be `docs/craft/specs/` (e.g. a project may use `docs/superpowers/specs/` and `docs/superpowers/plans/`).

Display the found specs to the user. If more than 3-4 are found, ask which are in scope before reading them all.

Also check `docs/craft/features/<slug>/` for:
- The most recent `YYYY-MM-DD-<slug>-state.md`, if any
- `overview.md`, if it exists

## Step 3 — Read: code first, specs second

Read the confirmed implementation files first. Code is the source of truth — specs describe intent, code describes reality.

Then read the relevant specs for context on decisions and rationale that are not visible from the code alone (e.g. why A was chosen over B, what was explicitly deferred).

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
- path/x.py - responsibility
- path/y.py - responsibility

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
- features/<slug>/overview.md
- [2-3 most critical implementation files - the ones a new session would need open to make good decisions]

## Related Specs
- specs/YYYY-MM-DD-<topic>-design.md - still relevant: <which parts>
- specs/YYYY-MM-DD-<topic>-design.md - superseded
```

Set `Supersedes:` to the filename of the previous state file (not a full path).

## Step 6 — Do not delete old state files

Keep all dated state files for history. The newest one is authoritative. Older ones stay as an audit trail.

## Step 7 — Offer to update `overview.md`

After writing the state file, ask:

> Any new canonical decisions to append to `features/<slug>/overview.md`?

`overview.md` is different from a state file. It is a small, stable list of canonical decisions and guidelines that accumulated across brainstorming sessions. It is not dated — it evolves in place. Examples of entries:

- "Custom Assistant is the main building block. Assistant is a special-case Custom Assistant with no global config."
- "All new features must gate behind a FeatureFlagName entry."

If `overview.md` does not exist yet and the user wants to add entries, create it with this shape:

```markdown
# <Feature Name> - Overview

Canonical decisions and long-lived guidelines for <feature>. Short, stable, evolves in place. Every entry traces back to a spec or a session.

## Core Decisions
- <Decision>. Source: specs/YYYY-MM-DD-<topic>-design.md

## Guidelines
- <Guideline>.

## Open Questions
- <Question>.

## History
- YYYY-MM-DD: added <decision> (source: spec or session).
```

When appending to an existing `overview.md`, append new entries under the right section and add a `History` line. Do NOT rewrite existing entries unless the user explicitly asks.

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
| "I will delete the old state file" | No. Keep all for history. |
| "I will write this without reading code" | No. Specs drift. Code is truth. Read code before specs. |
| "The spec had 5 open questions, I will list them all" | Only list ones still open today. Answer the rest from current code. |

## Integration

- Paired with `craft:implement`, which prompts (does not auto-invoke) this skill at finish.
- Feeds brainstorming sessions: the user should start by loading the files listed in "Load This In Next Session" instead of the full spec pile.
