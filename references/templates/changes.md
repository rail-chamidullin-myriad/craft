# <Topic> - <YYYY-MM-DD>

**Branch:** <branch-name>  (run `git log --oneline <main>..<branch>` for commits)
**Spec:** specs/<file>.md  (omit this line if from-conversation mode)

**Hard cap: 2k chars (~500 tokens). Above that, scope is wrong - the file is trying to be a snapshot.**

## What shipped

1-2 short paragraphs. Plain language, caveman-style. Cover:
- The core change in one sentence.
- Anything non-obvious a future reader would miss from `git log` + code: a tricky decision, a workaround, a constraint that surfaced.
- Skip restating file paths or function names - that's code.

## Gaps left open

Optional. One line each. Only items not captured as code TODOs.

---

**Filename:** `changes/YYYY-MM-DD-<short-topic>.md`. Same-day collision: append `b`, `c`, etc.

**Invariants:**
- Immutable once written. If reality diverged later, write a new entry; do not edit.
- Skip the file entirely when the session shipped only what `git log` already captures (typo fix, dep bump, mechanical refactor). The default for routine sessions is no file.
- No architecture dumps, no key-files lists, no commit-by-commit logs - those were the snapshot bloat vectors.
