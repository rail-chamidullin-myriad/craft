---
name: code-review
description: Run a fast, single-pass code review of a GitHub PR or the current branch. Outputs only three things — risk score (low/medium/high), code quality score (0-100), and the issues that score >= 80 on importance. Use when the user says "/code-review", "/code-review <PR#>", "code review", "quick review", "fast review", "lightweight code review", "tldr review", or asks for a code review and wants something shorter than the multi-agent reviewers shipped by other plugins. Prefer this over deeper review tools whenever the user signals they want speed, brevity, or just the headline risks.
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

## What to check

Two passes over the diff, in order:

**Pass 1 — bug check.** For each non-trivial code change, ask:
- Does it do what the PR title/description claims?
- Are there off-by-one, null/None, async/await, transaction-boundary, or type-confusion bugs?
- Does any change widen a public contract, alter a failure path, or affect shared state?
- Are tests covering the new logic? (Existence is enough — don't grade test quality here.)

**Pass 2 — description match.** Read the PR description (or commit messages, in branch mode) and confirm the diff actually delivers what's described. Flag mismatches as issues. If the description is empty or generic ("misc fixes"), that itself is a finding.

You do NOT need to: rewrite the code, suggest style nits, audit every test fixture, score test coverage on a rubric, or list every observation. If an issue wouldn't make you push back on the PR, leave it out.

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

For every concrete issue you'd consider raising, assign an **importance** score 0-100:

- **90-100** — bug, security issue, data-loss risk, contract violation, or "this PR doesn't do what its description says."
- **80-89** — meaningful correctness or maintainability concern: load-bearing assumption that's wrong, missing test for a non-trivial branch, fragile coupling that will break next quarter.
- **60-79** — solid suggestion but the PR is fine without it.
- **<60** — nit, style, taste. Drop these.

**Only show issues scoring 80 or higher.** If there are none, say so explicitly — "no issues at or above importance 80" — don't pad with lower-importance items.

For each shown issue:
- One line stating the problem with `file:line` if applicable.
- One line on why it matters (the *why*, not "this is wrong").
- Importance number in brackets at the end.

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
