---
name: implement
description: Execute a feature in the current session without writing an implementation plan file. Works in two modes - from-spec (read a design spec from disk and implement it) or from-conversation (implement directly from the current chat, skipping the spec step when the user already knows exactly what to build).
---

# Implement (craft)

Direct implementation in the main session loop, with no plan-on-disk. Discipline comes from TDD cadence, per-task commits, and a TodoWrite ledger - not a markdown plan file.

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

**Asking questions:** Ask when scope, design, or assumptions shift mid-work. Otherwise execute. If code disagrees with the spec, if an API behaves differently than expected, or if a new edge case appears, pause and check - don't guess.

**Default artifact root:** `docs/craft/`. A project's `CLAUDE.md` (or equivalent project instructions file) may override this path. If overridden, use the override everywhere.

**Announce at start:**
- from-spec: "I'm using the craft:implement skill to execute this spec."
- from-conversation: "I'm using the craft:implement skill in from-conversation mode."

## When to use

- The user has a spec file ready → from-spec mode.
- The user knows exactly what to build and wants to skip the spec → from-conversation mode.

## The Process

### 1. Set up an isolated worktree

Before touching any code, set up a git worktree so the implementation is isolated from the current workspace. This prevents accidental contamination of unrelated work, makes it safe to abandon a session, and allows parallel sessions on other branches.

**Directory selection (priority order):**
1. If `.worktrees/` exists at the repo root, use it.
2. Otherwise, check the project's `CLAUDE.md` for a documented worktree directory preference.
3. Otherwise, default to creating `.worktrees/` at the repo root. Confirm with the user before creating the directory itself if it does not exist.

**Verify `.worktrees/` is git-ignored before creating the worktree:**

```bash
git check-ignore -q .worktrees
```

If the directory is not ignored, add `.worktrees/` to `.gitignore` and commit that change before proceeding. This prevents accidentally tracking worktree contents.

**Branch naming:** If the project's `CLAUDE.md` documents a branch-name convention, follow it. Otherwise, default to `<type>/<slug>` where `<type>` is one of `feat`, `fix`, `chore`, `refactor` and `<slug>` is a short kebab-case descriptor (e.g. `feat/doc-review-page-limit`).

**Create the worktree and branch:**

```bash
git worktree add .worktrees/<branch-name> -b <branch-name>
cd .worktrees/<branch-name>
```

Work entirely inside the worktree for the remainder of the session - all file paths, commits, and verification commands run from there.

**Run project setup and verify a clean baseline.** Auto-detect based on files present (`package.json` → `npm install`, `pyproject.toml` → `uv sync` or `poetry install`, `Cargo.toml` → `cargo build`, `go.mod` → `go mod download`, etc.). Then run the project's test command to confirm the baseline is green before making changes. If baseline tests fail, report the failures and ask whether to proceed or investigate - do not silently continue.

"Baseline green" means no *pre-existing* failures before this session begins. The red tests you will write in step 4 for upcoming tasks are expected to fail until you implement them - that is the point of the TDD cadence, not a baseline regression.

**Opt-out:** If the user explicitly says "stay in the current workspace" or "no worktree", skip this step and proceed in the current tree. Do NOT silently skip.

### 2. Load the design (mode-dependent)

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

**Scope heuristic - switch to brainstorm if ANY of these are true:**
- The work touches 3+ files across different concerns (not 3 files in the same module).
- It requires architectural decisions (new abstractions, cross-cutting patterns, public API shape).
- You find yourself inferring more than directly instructed - your restate bullets contain guesses the user didn't explicitly make.
- The user cannot restate the scope in one sentence when asked.

from-conversation fits small, bounded, single-concern changes where the whole picture fits in a few bullets and nothing is being invented.

### 3. File-structure pre-flight (inline, no file)

Before writing code, state:
- Files to create (exact paths) and the responsibility of each.
- Files to modify (exact paths, approximate line ranges when relevant) and what changes.
- Files that deliberately stay untouched.

Design units with clear boundaries and one clear responsibility per file. Prefer smaller, focused files; follow established patterns when working in existing code.

Present this as a short bullet list, get user confirmation, then proceed.

### 4. Build the task ledger in TodoWrite

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

### 5. Execute in the main session loop

- Work one task at a time. Mark `in_progress` before starting, `completed` when done.
- Commit after each task, not at the end. Follow the project's commit conventions if documented in `CLAUDE.md`.
- Stop and ask when anything changes the picture - missing dependency, failing verification, unclear instruction, code disagreeing with the spec, an API behaving differently than expected, or a new edge case that the design did not account for. Do not guess.
- Never start implementation on `main` or `master` without explicit consent.

### 6. When to dispatch a subagent (exception, not default)

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

### 7. Pre-completion self-check (inline, not a document review)

Before declaring done:
- **Spec coverage** - every requirement in the spec maps to implemented code. List any gaps.
- **No placeholders** - no `TODO`, stub returns, `pass`-only bodies, "handle later" comments that were not explicitly agreed.
- **Name and type consistency** - method signatures, property names, and types match across files. A function called one thing in one place and another elsewhere is a bug.
- **No silenced warnings or type-check escape hatches** (e.g. `# type: ignore`, `@ts-ignore`, `eslint-disable`, `#[allow(...)]`) added to make the build pass. If the tool complains, fix the root cause.
- Self-review finds some classes of issues; a genuine review pass finds others. Do both.

### 8. Verify

Run the project's verification commands - the exact shortcuts depend on the project. Check the project's `CLAUDE.md` or README for documented ones; otherwise use whatever the project conventionally runs:

- Format + lint (e.g. `ruff`, `prettier`, `gofmt`, `rustfmt` - whatever the project uses).
- Type-check if the project has one (e.g. `pyright`, `tsc`, `mypy`).
- Test suite, scoped to the areas touched.

Report status: what changed, what is verified, anything still open.

### 9. Write task log (always)

Write a dated task log to `docs/craft/tasks/YYYY-MM-DD-<topic>.md` (create the directory with `mkdir -p` if needed). Use the topic slug from the spec filename when available, otherwise ask the user.

Template:

```markdown
# <Topic> - Implementation Tasks (YYYY-MM-DD)

**Branch:** <branch-name>
**Mode:** from-spec | from-conversation
**Spec:** specs/YYYY-MM-DD-<topic>-design.md (or "none - from-conversation mode")
**Summary:** One or two sentences on what this session accomplished. In from-conversation mode, include the agreed scope from step 1 here so the log is self-contained.

## Completed
- [x] Task 1 name
  - Files: path/to/module.<ext>, path/to/other.<ext>
  - Tests: path/to/module_tests.<ext>
  - Commit: <sha>
- [x] Task 2 name ...

## Deferred / Open
- Item not in scope this session (or "none")

## Verification
- format + lint - pass
- type-check - pass (or "n/a")
- tests - N passed
```

Pull task list and commit SHAs from TodoWrite and `git log`. Keep it short - this is a pointer, not an essay.

### 10. Offer feature-state and overview updates (on demand)

After writing the task log, ask the user:

> Update feature state? This writes a dated snapshot to `features/<feature>/YYYY-MM-DD-<slug>-state.md` via the craft:feature-state skill.

If yes, invoke `craft:feature-state`. If no, skip.

Then ask:

> Any new canonical decisions to append to `features/<feature>/overview.md`? (e.g., "Custom Assistant is the main building block; Assistant is a special case.")

If yes, append to the overview file. If no, skip. Do NOT mass-rewrite overview.md - only append new entries under the appropriate section.

### 11. Worktree hand-off

Close the session with a single status line plus neutral next-step options. Do NOT recommend a default - worktree cleanup conventions vary by user, and any built-in default will be wrong for most of them.

Send this as the final message:

> "Implementation is complete on `<branch>` at `.worktrees/<branch>`. Commits are already visible from the main checkout.
>
> What do you want to do next?
>
> 1. Remove the worktree now (useful if you want to check out `<branch>` in the main checkout to review/rebase/squash in your IDE).
> 2. Push from the worktree, keep it open (useful if you expect PR review feedback and want to avoid re-running project setup).
> 3. Leave it - you'll clean up later.
>
> Or tell me what you want."

If the user picks option 1:

```bash
# Run from the main checkout, NOT from inside the worktree.
git worktree remove .worktrees/<branch-name>
git worktree prune
```

Do NOT delete the local branch - the user still needs it for review and push. `git worktree remove` refuses to delete a worktree with uncommitted changes. Don't reach for `--force` unless the user has verified nothing is lost.

If the user picks option 2 or 3, or gives custom instructions, follow those and stop.

**Background (only mention if the user asks):** commits are visible from every worktree because worktrees share `.git`. A branch can only be checked out in one worktree at a time, so removing the feature worktree is what frees the main checkout to check out the feature branch.

## Red Flags

| Thought | Reality |
|---------|---------|
| "Let me write a plan file first" | No. Use TodoWrite. User workflow explicitly disables plan files. |
| "I will dispatch subagents for everything to go faster" | No. Main session is the default. Subagents are the exception. |
| "The spec has a gap but I will just pick something reasonable" | Stop. Ask the user. |
| "The spec says `TBD` but I will infer what was meant" | Stop. Ask the user. |
| "Self-review was clean so it is good" | Run the actual verifications (format/lint, type-check, tests). |
| "I will skip the failing-test step to save time" | No. The failing test proves the test exercises the change. |
| "One big commit at the end is cleaner" | No. Commit per task. |

## Integration

- Uses `TodoWrite` for task tracking.
- Use whatever commit, test, and lint conventions the project defines (check its `CLAUDE.md` or equivalent for shortcuts).
