---
name: implement
description: Execute a feature in the current session without writing an implementation plan file. Works in two modes - from-spec (read a design spec from disk and implement it) or from-conversation (implement directly from the current chat, skipping the spec step when the user already knows exactly what to build). Replaces superpowers:writing-plans and superpowers:executing-plans.
---

# Implement (craft)

Direct implementation in the main session loop, with no plan-on-disk. Keeps the disciplines from `superpowers:writing-plans` and `superpowers:executing-plans` but drops the paperwork.

## Modes

**from-spec** (default when the user points at a spec file):
- A design spec exists on disk, typically in `docs/craft/specs/YYYY-MM-DD-<topic>-design.md`.
- Read the spec first, then implement.

**from-conversation** (use when the user describes the work inline and skips the spec step):
- No spec file. The design is whatever has been agreed in the current conversation.
- Before starting, restate the understood scope in 3-5 bullet points and ask the user to confirm or correct. This replaces the "critical spec review" step.
- Write a short summary of the agreed scope into the task log at finish (step 8), so the session is still traceable.
- If the agreed scope is large enough that it would benefit from a spec, say so and offer to switch to `craft:brainstorm` instead. Proceed only if the user confirms they want to skip the spec.

**Principles:** DRY. YAGNI. KISS. TDD. Frequent commits.

**Default artifact root:** `docs/craft/`. A project's `CLAUDE.md` or `CLAUDE.local.md` may override this path. If overridden, use the override everywhere.

**Announce at start:**
- from-spec: "I'm using the craft:implement skill to execute this spec."
- from-conversation: "I'm using the craft:implement skill in from-conversation mode."

## When to use

- The user has a spec file ready → from-spec mode.
- The user knows exactly what to build and wants to skip the spec → from-conversation mode.

Do NOT use `superpowers:writing-plans` or `superpowers:executing-plans` in this repo - they are explicitly disabled by user workflow overrides.

## The Process

### 0. Set up an isolated worktree

Before touching any code, set up a git worktree so the implementation is isolated from the current workspace. This prevents accidental contamination of unrelated work, makes it safe to abandon a session, and allows parallel sessions on other branches.

- **REQUIRED SUB-SKILL:** Use `superpowers:using-git-worktrees` to create the worktree and a feature branch.
- If the user explicitly opts out ("stay in the current workspace", "no worktree"), skip this step and proceed in the current tree. Do NOT silently skip.
- Work entirely inside the worktree for the remainder of the session. All file paths, commits, and verification commands run from there.

### 1. Load the design (mode-dependent)

**from-spec:**
- Read the spec end to end.
- List any ambiguities, gaps, or conflicting requirements.
- Scope check: if the spec spans multiple independent subsystems, propose splitting before coding. Each chunk should be independently working and testable.
- If the spec has unresolved entries under `Open Questions`, treat them as blockers. Ask the user before coding anything that depends on them.
- Surface concerns to the user and wait for answers on anything load-bearing. Do not guess.

**from-conversation:**
- Restate your understanding of the agreed scope in 3-5 bullet points. Include goal, key files to touch, and any explicit constraints.
- Flag anything you are inferring rather than directly instructed.
- Ask the user to confirm or correct before proceeding. Do not start coding from your first interpretation - the confirmation step is the entire safety net in this mode.
- If the scope feels large or exploratory mid-restate, suggest switching to `craft:brainstorm` to produce a spec. Proceed only if the user waves you on.

### 2. File-structure pre-flight (inline, no file)

Before writing code, state:
- Files to create (exact paths) and the responsibility of each.
- Files to modify (exact paths, approximate line ranges when relevant) and what changes.
- Files that deliberately stay untouched.

Design units with clear boundaries and one clear responsibility per file. Prefer smaller, focused files; follow established patterns when working in existing code.

Present this as a short bullet list, get user confirmation, then proceed.

### 3. Build the task ledger in TodoWrite

- Decompose work into bite-sized tasks. Each task produces a self-contained change that makes sense independently.
- Use TodoWrite as the ledger. No markdown plan file on disk.
- Per task, the step cadence is:
  - Write the failing test.
  - Run it and verify it fails for the right reason.
  - Implement the minimal code to make the test pass.
  - Run the tests and verify they pass.
  - Commit.
- Each step is one action (2-5 minutes). If a step feels bigger than that, split it.
- Skip the TDD cadence only when it genuinely does not apply (pure config change, UI tweak that cannot be unit-tested). Say so explicitly when you skip.

### 4. Execute in the main session loop

- Work one task at a time. Mark `in_progress` before starting, `completed` when done.
- Commit after each task, not at the end. See `@.claude/skills/commit/SKILL.md` (`/commit`) for commit conventions.
- Stop on any blocker - missing dependency, failing verification, unclear instruction. Ask the user; do not guess.
- Never start implementation on `main` or `master` without explicit consent.

### 5. When to dispatch a subagent (exception, not default)

Default is main-session execution. Dispatch a subagent only when ALL of the following hold:
- The task is isolated and well-scoped (one or two files, no cross-cutting changes).
- The task does not change architecture, public APIs, or shared conventions.
- The work is mechanical enough that a fresh-context agent can do it correctly.

When dispatching:
- Provide the full task text and the specific surrounding context. Never tell the subagent to "read the spec" or "figure it out".
- Require the subagent to ask clarifying questions before implementing if anything is ambiguous.
- Accept one of four status reports: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, `BLOCKED`. Handle each:
  - `DONE` - proceed to review.
  - `DONE_WITH_CONCERNS` - read concerns; address correctness/scope issues before review, note observations and proceed.
  - `NEEDS_CONTEXT` - supply the missing context and re-dispatch.
  - `BLOCKED` - reassess: provide more context, or use a more capable model, or break the task smaller, or escalate.
- Never dispatch two implementation subagents in parallel on overlapping files.
- Model selection: cheap model for mechanical single-file work, standard for multi-file integration, most capable for judgment calls.

### 6. Pre-completion self-check (inline, not a document review)

Before declaring done:
- **Spec coverage** - every requirement in the spec maps to implemented code. List any gaps.
- **No placeholders** - no `TODO`, stub returns, `pass`-only bodies, "handle later" comments that were not explicitly agreed.
- **Name and type consistency** - method signatures, property names, and types match across files. A function called one thing in one place and another elsewhere is a bug.
- **No pyright/type-ignore comments** - if types do not check, fix the root cause.
- Self-review finds some classes of issues; a genuine review pass finds others. Do both.

### 7. Verify

- Run `just f` (format + lint).
- Run `just p` (pyright).
- Run the relevant `just t-*` target(s) for the areas touched.
- Report status: what changed, what is verified, anything still open.
- Do NOT invoke `superpowers:finishing-a-development-branch`.

### 8. Write task log (always)

Write a dated task log to `docs/craft/tasks/YYYY-MM-DD-<topic>.md`. Use the topic slug from the spec filename when available, otherwise ask the user.

Template:

```markdown
# <Topic> - Implementation Tasks (YYYY-MM-DD)

**Branch:** <branch-name>
**Mode:** from-spec | from-conversation
**Spec:** specs/YYYY-MM-DD-<topic>-design.md (or "none - from-conversation mode")
**Summary:** One or two sentences on what this session accomplished. In from-conversation mode, include the agreed scope from step 1 here so the log is self-contained.

## Completed
- [x] Task 1 name
  - Files: path/a.py, path/b.py
  - Tests: path/a_tests.py
  - Commit: <sha>
- [x] Task 2 name ...

## Deferred / Open
- Item not in scope this session (or "none")

## Verification
- `just f` - pass
- `just p` - pass
- `just t-<target>` - N passed
```

Pull task list and commit SHAs from TodoWrite and `git log`. Keep it short - this is a pointer, not an essay.

### 9. Offer feature-state and overview updates (on demand)

After writing the task log, ask the user:

> Update feature state? This writes a dated snapshot to `features/<feature>/YYYY-MM-DD-<slug>-state.md` via the craft:feature-state skill.

If yes, invoke `craft:feature-state`. If no, skip.

Then ask:

> Any new canonical decisions to append to `features/<feature>/overview.md`? (e.g., "Custom Assistant is the main building block; Assistant is a special case.")

If yes, append to the overview file. If no, skip. Do NOT mass-rewrite overview.md - only append new entries under the appropriate section.

Finally, hand control back to the user with a short summary.

## Red Flags

| Thought | Reality |
|---------|---------|
| "Let me write a plan file first" | No. Use TodoWrite. User workflow explicitly disables plan files. |
| "I will dispatch subagents for everything to go faster" | No. Main session is the default. Subagents are the exception. |
| "The spec has a gap but I will just pick something reasonable" | Stop. Ask the user. |
| "The spec says `TBD` but I will infer what was meant" | Stop. Ask the user. |
| "Self-review was clean so it is good" | Run the actual verifications (`just p`, `just t-*`). |
| "I will skip the failing-test step to save time" | No. The failing test proves the test exercises the change. |
| "One big commit at the end is cleaner" | No. Commit per task. |

## Integration

- Uses `TodoWrite` for task tracking.
- Commit via `/commit` skill conventions.
- Test/lint via `/test` and `/lint` skills or the `just` shortcuts in `CLAUDE.local.md`.
- Path-scoped rules in `.claude/rules/` auto-load when you touch matching files - do not duplicate their content here.
