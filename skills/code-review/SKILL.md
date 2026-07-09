---
name: code-review
description: Run a fast code review of a GitHub PR or the current branch - single-pass by default, parallel finder lenses with verification on large diffs. Outputs only three things - risk score (low/medium/high), code quality score (0-100), and the issues that score >= 80 on both importance and confidence. Use when the user says "/code-review", "/code-review <PR#>", "code review", "quick review", "fast review", "lightweight code review", "tldr review", or asks for a code review and wants something shorter than the multi-agent reviewers shipped by other plugins. Prefer this over deeper review tools whenever the user signals they want speed, brevity, or just the headline risks.
---

# Code Review

A deliberately minimal code review. The goal is to tell the user, in under a screen of text:

1. How risky this change is.
2. How clean the code is.
3. Only the issues that actually matter.

The user already has access to slower, more thorough reviewers. This skill exists because those are too long for everyday use.

**The output is about the review, not the PR.** The user can read the PR description themselves — don't recapitulate what the PR does. Every line should carry review signal: a score, a finding, or the reasoning behind a score.

## Inputs

Argument handling:

- If the user passes a number (e.g. `/code-review 5567`), treat it as a GitHub PR number.
- If they pass nothing, review the current branch's diff against `master` (or `main` if `master` doesn't exist).
- If they pass a branch name, diff that branch against master.

Fetch sources:

- PR mode: `gh pr view <num>` for title/description, `gh pr diff <num>` for the diff.
- Branch mode: `git log --oneline master..HEAD` for commits, `git diff master...HEAD` for the diff.

If the diff is large, page through it rather than truncating — every file matters for the risk score, but you do NOT need to deep-read fixtures, generated code (`*.gen.ts`, OpenAPI JSON), translation JSON, large HTML test fixtures, or lockfile changes. Scan their existence so the risk score reflects them, but don't quote them.

### Fetch rules — no local artifacts

Every fetch is read-only against the remote. Nothing should land in the working tree or in `git branch` for the user to clean up after.

- PR mode: only `gh pr view <num>`, `gh pr diff <num>`, and `gh api` calls. For full-file context at the PR's head, use `gh api repos/{owner}/{repo}/contents/<path>?ref=<sha>` or `gh pr view <num> --json files`.
- Branch mode: only `git log` and `git diff` against `master`. Do not switch branches or create new ones.
- Forbidden — these all create artifacts: `gh pr checkout`, `git fetch origin pull/<num>/head`, `git switch -c`, `git checkout -b`, `git worktree add`.
- Red flag: if you find yourself wanting to "just check out the PR to read a file," stop and use `gh api` instead.

## What to check

Two passes over the diff, in order:

**Pass 1 - bug hunt.** Sweep every non-trivial code change through the lenses below. Read beyond the diff hunk when a lens calls for it: the surrounding function, a field's doc comment, a sibling entry. The diff shows what changed; bugs usually live in how the change meets the code around it.

**Pass 2 - description match.** Read the PR description (or commit messages, in branch mode) and confirm the diff actually delivers what's described. A mismatch is a finding only when the code *behaves* differently from what the description claims - imprecise wording in a description, docstring, or comment with no behavioral consequence is maintainability, usually below the reporting bar. If the description is empty or generic ("misc fixes"), that itself is a finding.

You do NOT need to: rewrite the code, suggest style nits, audit every test fixture, score test coverage on a rubric, or list every observation. If an issue wouldn't make you push back on the PR, leave it out.

### Lenses

The pass-1 checklist in single-pass mode; one finder each in deep mode. Skip lenses with no surface in the diff.

- **Correctness** - off-by-one, null/None, async/await, type confusion, transaction boundaries, races and idempotency under retry/redelivery, altered failure paths, widened public contracts. Existence of tests for new logic counts here - don't grade test quality.
- **Security** - untrusted input reaching SQL/shell/HTML/LLM prompts, missing authz on new endpoints, secrets or PII in logs and error messages.
- **Data quality** - migrations vs existing rows, partial-failure states, lost writes, timezone/encoding handling, wrong-but-plausible values persisted.
- **Performance** - N+1 queries, unbounded fan-out or batch sizes, missing limits/pagination on new queries, slow work inside locks.
- **Consistency** - for every added config block, enum entry, or wiring registration: read the field's doc comment and one sibling entry, and flag deviations (a value contradicting the documented contract, a companion field siblings set but this block omits, a registration missing from one place siblings appear in).

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

This is about how clean the code *itself* is, independent of risk. Use the full range — don't cluster everything at 70-85.

- **90-100** — idiomatic, follows project conventions, well-named, tested, comments only where needed.
- **70-89** — solid but with rough edges (dead code, mild over-engineering, redundant checks, one or two awkward abstractions).
- **50-69** — works but noticeably messy: copy-paste, unclear naming, missing tests, defensive code for impossible cases.
- **<50** — meaningful structural problems: bugs latent in the design, broken layering, tests that don't test anything.

State the score on one line with a one-clause reason.

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

## Behavioural notes

- Do not write a long preamble. Start with the heading.
- Do not score the same issue twice (e.g., once as a bug, once as a test gap). Pick the highest-importance framing.
- If the PR is purely generated code (FE types, OpenAPI), say so and skip the issue list entirely.
- If the diff is empty, say so and stop — don't invent issues.
- Never refuse to score. Even messy, hard-to-grade changes get a number; that's the point of forcing a score.
