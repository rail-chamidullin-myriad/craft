---
name: implement
description: Execute a feature end-to-end in the current session, without writing an implementation plan file. Use this whenever the user says "implement this", "build X", "let's code this up", "execute the spec", "carry out the design", "do the work", points at a spec file, or asks to ship/deliver something that has been designed. Two modes — from-spec (reads a design spec from craft memory and implements it) or from-conversation (implements directly from the current chat, skipping the spec step when the user already knows exactly what to build). Sets up a git worktree for isolation, uses TodoWrite as the task ledger, follows TDD cadence (failing test → impl → run → commit), and optionally prompts to update feature state at finish.
---

# Implement (craft)

Direct implementation in the main session loop, with no plan-on-disk. Discipline comes from TDD cadence, per-task commits, and a TodoWrite ledger - not a markdown plan file.

## Modes

**from-spec** (default when the user points at a spec file):
- A design spec exists on disk at `<memory-root>/specs/YYYY-MM-DD-<topic>-design.md`.
- Read the spec first, then implement.

**from-conversation** (use when the user describes the work inline and skips the spec step):
- No spec file. The design is whatever has been agreed in the current conversation.
- Before starting, restate the understood scope in 3-5 bullet points and ask the user to confirm or correct. This replaces the "critical spec review" step.
- Include the agreed scope in the final hand-off summary (step 10), so the session is still traceable.
- If the agreed scope is large enough that it would benefit from a spec, say so and offer to switch to `craft:brainstorm` instead. Proceed only if the user confirms they want to skip the spec.

**Principles:** DRY. YAGNI. KISS. TDD. Frequent commits.

**Asking questions:** Ask when scope, design, or assumptions shift mid-work. Otherwise execute. If code disagrees with the spec, if an API behaves differently than expected, or if a new edge case appears, pause and check - don't guess.

**Memory:** this skill reads specs from craft memory. Path, layout, and invariants are defined once in the shared reference [`../../references/memory.md`](../../references/memory.md) — read it before any memory access. `<memory-root>` is used throughout this document to refer to the path defined there.

**Announce at start:**
- from-spec: "I'm using the craft:implement skill to execute this spec."
- from-conversation: "I'm using the craft:implement skill in from-conversation mode."

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
- **Surgical changes only.** Touch only what the task requires. Do not "improve" adjacent code, comments, or formatting on the way past. Match existing style even if you would write it differently. Remove imports/variables/functions that *your* changes orphaned; leave pre-existing dead code alone unless explicitly asked. Every changed line should trace directly to the user's request - if you cannot justify a hunk by pointing at the task, revert it.

### 6. When to dispatch a subagent (exception, not default)

Default is main-session execution — it has context a fresh agent does not, and course-correction is cheap here. Dispatch a subagent only for isolated, mechanical, single-concern work that does not change architecture or shared conventions. When you do dispatch, read `references/subagent-dispatch.md` for the full protocol (criteria, status-report handling, model selection).

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

### 9. Offer feature-state and overview updates (on demand)

Ask the user:

> Update feature state? This writes a new `features/<feature>/snapshots/YYYY-MM-DD.md` via the craft:feature-state skill.

If yes, invoke `craft:feature-state`. If no, skip.

Then ask:

> Any new canonical decisions to append to `features/<feature>/overview.md`? (e.g., "Custom Assistant is the main building block; Assistant is a special case.")

If yes, append to the overview file. If no, skip. Do NOT mass-rewrite overview.md - only append new entries under the appropriate section.

### 10. Worktree hand-off

Close the session with a single status line plus neutral next-step options. Do NOT recommend a default - worktree cleanup conventions vary by user, and any built-in default will be wrong for most of them.

Send this as the final message (substitute `<N>` from `git rev-list --count <branch> ^<base>` and the verification line from step 8):

> "Implementation is complete on `<branch>` at `.worktrees/<branch>` — `<N>` commits, verification: lint=pass type=pass tests=`<N>` passed. Commits are already visible from the main checkout.
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
| "I'll clean up this nearby code while I'm here" | No. Surgical changes only. Mention it; don't touch it. |
| "I'll match my preferred style instead of the existing one" | No. Match the surrounding code, even if you'd write it differently. |

## Integration

- Uses `TodoWrite` for task tracking.
- Use whatever commit, test, and lint conventions the project defines (check its `CLAUDE.md` or equivalent for shortcuts).
