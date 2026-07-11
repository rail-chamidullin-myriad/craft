---
name: code-review
description: Run a fast code review of a GitHub PR or the current branch - single-pass by default, parallel finder lenses with verification on large diffs. Outputs only three things - risk score (low/medium/high), code quality score (0-100) that weighs structure and reviewability alongside correctness, and the issues that score >= 80 on both importance and confidence. Use when the user says "/code-review", "/code-review <PR#>", "code review", "quick review", "fast review", "lightweight code review", "tldr review", or asks for a code review and wants something shorter than the multi-agent reviewers shipped by other plugins. Prefer this over deeper review tools whenever the user signals they want speed, brevity, or just the headline risks.
---

# Code Review

A deliberately minimal code review. The goal is to tell the user, in under a screen of text:

1. How risky this change is.
2. How clean and reviewable the code is.
3. Only the issues that actually matter.

The user already has access to slower, more thorough reviewers. This skill exists because those are too long for everyday use.

**The output is about the review, not the PR.** The user can read the PR description themselves — don't recapitulate what the PR does. Every line should carry review signal: a score, a finding, or the reasoning behind a score.

## Inputs

Argument handling:

- If the user passes a number (e.g. `/code-review 5567`), treat it as a GitHub PR number.
- If they pass nothing, review the current branch's diff against `master` (or `main` if `master` doesn't exist).
- If they pass a branch name, diff that branch against master.

Fetch sources:

- PR mode: `gh pr view <num>` for title/description, `gh pr diff <num>` for the diff, `gh pr view <num> --json commits` for the commit list (feeds the reviewability score).
- Branch mode: `git log --oneline master..HEAD` for commits, `git diff master...HEAD` for the diff.

If the diff is large, page through it rather than truncating — every file matters for the risk score, but you do NOT need to deep-read fixtures, generated code (`*.gen.ts`, OpenAPI JSON), translation JSON, large HTML test fixtures, or lockfile changes. Scan their existence so the risk score reflects them, but don't quote them.

### Fetch rules — no local artifacts

Every fetch is read-only against the remote. Nothing should land in the working tree or in `git branch` for the user to clean up after.

- PR mode: only `gh pr view <num>` (incl. `--json files,commits`), `gh pr diff <num>`, and `gh api` calls. For full-file context at the PR's head, use `gh api repos/{owner}/{repo}/contents/<path>?ref=<sha>`.
- Branch mode: only `git log` and `git diff` against `master`. Do not switch branches or create new ones.
- Forbidden — these all create artifacts: `gh pr checkout`, `git fetch origin pull/<num>/head`, `git switch -c`, `git checkout -b`, `git worktree add`.
- Red flag: if you find yourself wanting to "just check out the PR to read a file," stop and use `gh api` instead.

## What to check

Two passes over the diff, in order:

**Pass 1 - bug hunt.** Sweep every non-trivial code change through the lenses below. Read beyond the diff hunk when a lens calls for it: the surrounding function, a field's doc comment, a sibling entry. The diff shows what changed; bugs usually live in how the change meets the code around it.

**Pass 2 - description match.** Read the PR description (or commit messages, in branch mode) and confirm the diff actually delivers what's described. A mismatch is a finding only when the code *behaves* differently from what the description claims - imprecise wording in a description, docstring, or comment with no behavioral consequence is maintainability, usually below the reporting bar. If the description is empty or generic ("misc fixes"), that itself is a finding.

**Pass 3 - structure and reviewability.** Read the changed functions whole, not just the hunks, and read the commit list. Ask: did any function get longer or more deeply nested without extraction? Is there re-derived state, duplicated logic, or a predicate/assert smell? Does the diff braid pure moves/refactors with new logic so git can't tell them apart? Are the commits logically grouped and individually reviewable? This pass feeds the code-quality score - most of what it finds never clears the issue-reporting bar, and that is fine; the score carries it, not the issue list.

You do NOT need to: rewrite the code, audit every test fixture, score test coverage on a rubric, or list every observation as an issue. Style nits and structural rough edges shape the quality score but stay out of the issues list unless they clear the importance bar.

### Lenses

The pass-1 checklist in single-pass mode; one finder each in deep mode. Skip lenses with no surface in the diff.

- **Correctness** - off-by-one, null/None, async/await, type confusion, transaction boundaries, races and idempotency under retry/redelivery, altered failure paths, widened public contracts. Existence of tests for new logic counts here - don't grade test quality.
- **Security** - untrusted input reaching SQL/shell/HTML/LLM prompts, missing authz on new endpoints, secrets or PII in logs and error messages.
- **Data quality** - migrations vs existing rows, partial-failure states, lost writes, timezone/encoding handling, wrong-but-plausible values persisted.
- **Performance** - N+1 queries, unbounded fan-out or batch sizes, missing limits/pagination on new queries, slow work inside locks.
- **Consistency** - for every added config block, enum entry, or wiring registration: read the field's doc comment and one sibling entry, and flag deviations (a value contradicting the documented contract, a companion field siblings set but this block omits, a registration missing from one place siblings appear in).
- **Structure & reviewability** (feeds the quality score, rarely the issue list) - function-length growth, nesting, redundancy, re-derived state, predicate/assert smells, booleans that want to be types, moves braided with new logic, and commit grouping. In deep mode this finder reports observations for the score, not `file:line` bugs.

## Depth

Single pass in the main loop is the default and right for most PRs.

Switch to deep mode when the diff has 500+ changed lines of real code (excluding fixtures, generated files, lockfiles, translations) or the user asks for a deep/thorough review:

1. Launch parallel read-only finder subagents, one per applicable lens. Give each the PR number (or branch), the changed-file list, its lens, and the fetch rules above. Each returns candidate findings: `file:line`, a one-line summary, and a concrete failure scenario.
2. Verify every candidate yourself in the main loop: fetch the surrounding code and try to refute it. Drop what you refute.
3. Merge duplicates, then score survivors like any other issue.

Finders find; only the main loop scores and reports.

## Scoring

### Risk (low / medium / high)

Risk is **about blast radius if something goes wrong**, not about how clean the code is. A perfectly-written change to the auth middleware is still high-risk. A messy script in `research/` is low-risk.

Guidance:

- **low** — isolated change, easy to revert, no shared state. Examples: one new source module, a doc tweak, a self-contained script, a test-only change.
- **medium** — touches code other features rely on, or a shared library, or has a migration. Reversible but with effort. Affects one team's product surface.
- **high** — touches auth, billing, data migrations, core infra, shared types used cross-app, or anything that can't be cleanly rolled back. Also: large diffs (500+ lines of real code, ignoring fixtures) almost always default to at least medium.

State the risk on one line with a one-clause reason. No paragraph.

### Code quality (0-100)

How clean the code *itself* is, independent of risk. This is the score that drifts to a lazy ~80: a change with no bugs *feels* fine, so the number defaults high. Fight that directly. **The score is derived from concrete observations, not an overall impression, and "no bugs found" is not evidence of cleanliness** - it says nothing about whether the code is readable, simple, or reviewable.

Score these five dimensions internally, each against its anchors, then **take the overall as the lowest one or two, not the average.** Averaging re-centers everything to 80 and lets one clean dimension mask a real structural problem. One weak dimension caps the grade.

- **Structure** - function/method length, single responsibility, nesting depth, parameter counts. Judge the *delta*: a change that lengthens an already-long function, deepens nesting, or adds a branch without extracting is a structural regression *even though the mess predates the PR*. The PR earns no pass for "it was already long" - it had the chance to not make it worse.
- **Readability** - naming, control-flow clarity, whether a cold reader follows the change without the author's context.
- **Simplicity** - redundancy and needless indirection: dead code, re-derived or double-computed state, duplicated logic, a boolean flag that wants to be a type, or a predicate that returns a bool and then forces the caller to re-assert the same state (a helper that internally proves `x is not None` but returns `bool`, so the caller re-checks with `require_not_none`/an assert - the type system should carry that guarantee, not a runtime re-check).
- **Reviewability** - can a human actually review this? From the diff and the commit list: (a) does the diff braid pure moves/refactors with new logic so git can't separate them, forcing the reviewer to re-read unchanged code line by line - estimate the moved-or-refactored vs genuinely-new ratio, and a high ratio with no separation is a hit; (b) are the commits logically grouped and individually reviewable, or one blob a reviewer can't step through commit-by-commit.
- **Convention fit** - follows the project's established idioms and patterns.

Overall anchors:

- **90-100** - idiomatic, well-named, tested, appropriately factored, comments only where needed, and cleanly reviewable. Score here *only* if you found zero structural, simplicity, or reviewability defects.
- **70-89** - solid with rough edges: one or two awkward abstractions, mild redundancy, a slightly-too-long function, minor convention drift.
- **50-69** - works but noticeably messy: a long function made longer, re-derived state, a predicate/assert smell, moves braided with new logic so the diff is hard to review, unclear naming, or missing tests for new logic.
- **<50** - meaningful structural problems: broken layering, a design that invites bugs, tests that don't test anything.

You must be able to point at named defects to justify any score below 90; a bare "feels solid, 85" is exactly the failure this rubric exists to prevent. Structural, simplicity, and reviewability defects set the score even when they never rise to a listed issue - the grade and the issue list are separate axes.

State the score on one line with a one-clause reason that names the dimension that capped it and the specific defect behind it.

## Issue scoring

Every candidate issue gets two scores, 0-100 each. **Show only issues with both at 80 or higher.**

**Importance** - how much it matters:

- **90-100** - bug, security issue, data-loss risk, contract violation, or "this PR doesn't do what its description says."
- **80-89** - meaningful correctness or maintainability concern: load-bearing assumption that's wrong, missing test for a non-trivial branch, fragile coupling that will break next quarter.
- **60-79** - solid suggestion but the PR is fine without it.
- **<60** - nit, style, taste. Drop these.

**Confidence** - how sure you are the issue is real. Before scoring it, re-read the actual code around the finding, not just the diff hunk. Any bug-class claim must name a concrete failure scenario: the inputs or state that trigger it and the wrong outcome. Without a scenario, confidence caps at 50.

Never report:

- Pre-existing issues on lines the PR didn't touch.
- Anything a linter, type-checker, or CI would catch.
- Intentional behavior changes that serve the PR's stated purpose.
- Docstring/comment imprecision with no behavioral consequence.

If nothing clears both bars, say so explicitly - "no issues at or above importance 80" - don't pad with lower-scoring items.

For each shown issue:
- One line stating the problem with `file:line` if applicable.
- One line on the failure scenario or why it matters (the *why*, not "this is wrong").
- Importance number in brackets at the end. Confidence is a filter, not part of the output.

## Output format

Use exactly this structure. Keep it tight — the whole output should fit on one screen.

```
## Code Review: <PR title or branch name>

**Risk:** <low|medium|high> — <one-clause reason>
**Code quality:** <0-100> — <one-clause reason>

**Summary.** <One short line stating what the review checked and what it found. Read like a status report, not an essay. Examples: "No issues found. Checked for bugs and CLAUDE.md compliance." / "Two correctness issues found, see below." / "Description matches diff; no bugs spotted." Do NOT recap the PR. Do NOT justify the scores — that's what the one-clause reasons after Risk/Quality are for.>

**Issues (importance ≥ 80):**

- `<path>:<line>` — <problem>. <why it matters> [<score>]
- `<path>:<line>` — <problem>. <why it matters> [<score>]

(or: "No issues at or above importance 80.")
```

No "Test coverage" section, no "Security" section, no "Verdict" section. The risk + quality scores already encode that.

**Do not summarise the PR's contents.** Lines like "Adds X with 5 resource kinds, four list journals plus one single-page listing…" belong in the PR description, not in a review. If you catch yourself listing what the PR does, delete the line. The only acceptable PR-content reference is when it's load-bearing for a finding (e.g., "the diff matches the description" or "the description claims X but the diff does Y").

## Final step — offer to post the review

After printing the review, **always** ask the user whether to post it as a PR comment. Only when in PR mode (i.e., a PR number was passed or inferred). Skip in branch mode — there's no PR yet.

Phrase it as a one-line offer at the end, after the review output:

> Want me to post this as a comment on PR #<num>? (yes/no)

If the user says yes, post with:

```
gh pr comment <num> --body "$(cat <<'EOF'
<the review output, verbatim>
EOF
)"
```

Do not post automatically. Do not nag if they say no.

## Worked examples

**Example 1 — clean PR, low risk.**

```
## Code Review: [Feat] (HS) Add CY DPA source (#5567)

**Risk:** low — additive new source module, no changes to shared logic; FE/OpenAPI updates are generated.
**Code quality:** 88 — follows existing horizon-scanning conventions, well-tested with fixture replay; one redundant `require_not_none` and a couple of load-bearing lowercasing assumptions undocumented.

**Summary.** No issues found. Checked for bugs, description-vs-diff match, and CLAUDE.md compliance.

**Issues (importance ≥ 80):**

- No issues at or above importance 80.
```

**Example 2 — risky PR, mixed quality.**

```
## Code Review: rc-fix-auth-token-refresh

**Risk:** high — modifies session token refresh in auth middleware used by all services; rollback requires coordinated redeploy.
**Code quality:** 64 — works but the new retry loop swallows the underlying exception class, and the test only covers the happy path.

**Summary.** Two correctness issues found, see below. Description matches diff.

**Issues (importance ≥ 80):**

- `auth/middleware.py:142` — `except Exception` swallows the original `TokenExpiredError` and re-raises as a generic `AuthError`. This makes the very class of bug this PR fixes invisible if it recurs. [92]
- `auth/middleware.py:155` — no test exercises the refresh-failure branch this PR is built around. The fix's correctness rests entirely on manual reasoning. [85]
```

**Example 3 — correct and tested, but hard to review.** The kind of PR that used to score a reflexive ~85 because nothing was broken.

```
## Code Review: [Feat] (HS) Skip downstream LLM steps on formal-only updates (#8172)

**Risk:** medium — reshapes the always-run digestion path behind a dark flag; reversible via config.
**Code quality:** 66 — reviewability capped it: the diff braids a large verbatim method-extraction and a test-fixture reshuffle with ~90 lines of genuinely new logic, so a reviewer can't tell moved code from changed code; compounded by a gate predicate that returns a bool then makes the caller re-assert the same state via `require_not_none`, and an orchestrator function left longer, not shorter.

**Summary.** No issues at or above importance 80; the score is driven by structure and reviewability, not a bug. Behavior matches the description.

**Issues (importance ≥ 80):**

- No issues at or above importance 80.
```

## Behavioural notes

- Do not write a long preamble. Start with the heading.
- Do not score the same issue twice (e.g., once as a bug, once as a test gap). Pick the highest-importance framing.
- If the PR is purely generated code (FE types, OpenAPI), say so and skip the issue list entirely.
- If the diff is empty, say so and stop — don't invent issues.
- Never refuse to score. Even messy, hard-to-grade changes get a number; that's the point of forcing a score.
- A PR with no bugs is not automatically high quality. Score structure, simplicity, and reviewability on their own evidence; "nothing broke" says nothing about how clean or reviewable the change is. If the quality reason only restates "no bugs found," you skipped Pass 3.
- Structural, simplicity, and reviewability defects set the quality score even when they never become listed issues. The grade and the issue list are separate axes; a review can report zero issues and still score in the 60s.
