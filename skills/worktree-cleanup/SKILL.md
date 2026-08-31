---
name: worktree-cleanup
description: Audit and clean up stale local git worktrees and branches safely. Use when the user says "clean up worktrees", "clean up branches", "prune worktrees", "remove merged branches", "worktree cleanup", or asks to tidy a repo with many worktrees or branches. Audits first and shows a summary before removing anything. Removes only worktrees whose branch is merged or whose remote branch is gone (and, on request, unmerged worktrees that are clean and fully pushed). Never touches locked worktrees, uncommitted work, or unpushed commits.
---

# Worktree cleanup (craft)

Remove stale local worktrees and branches without destroying work. Audit first, get a go-ahead, then delete in small batches.

**Announce at start:** "I'm using the craft:worktree-cleanup skill."

**Principle:** deletion is destructive and other Claude sessions may be using worktrees live. Show the audit, wait for "go ahead", never remove a worktree with real work or a branch with unpushed commits.

## 1. Audit (read-only) and show a summary

Run these and present counts before deleting anything:

```bash
git worktree list                       # all worktrees; note "locked" ones
git branch --show-current               # never delete this
git branch --merged master | grep -vx master           # merged branches
git branch -vv | grep ': gone]'                          # remote branch deleted (PR landed)
```

Classify:
- **Deletable branches** = merged into master OR gone-upstream, AND not checked out in any worktree.
- **Removable worktrees** = branch merged or gone, AND working tree clean, AND not locked by a live session.
- **Keep** = live-locked, unmerged with work, or anything with uncommitted/unpushed changes.

**Live vs stale lock.** A lock is not proof of a running session; it can be left behind by a crashed one. The lock reason carries `pid <N> start <date>`. If that pid is dead, the lock is stale and the worktree follows the normal gate:

```bash
git worktree list --porcelain | grep -A1 '^worktree' | grep locked   # read the pid in each reason
kill -0 <pid> 2>/dev/null && echo live || echo stale                  # stale = safe to consider
```

Unlock a stale one before removing: `git worktree unlock <path>` (or `git worktree remove --force` handles it). When a lock has no pid, or you cannot tell, treat it as live and keep it.

## 2. Clean-detection: ignore sparse-checkout phantoms

Worktrees using sparse-checkout show thousands of phantom ` D` (deleted) entries in `git status` that are NOT real work. Real work is `M`, `A`, `R`, or `??` lines only. A worktree is clean when it has no such lines:

```bash
git -C <worktree> status --porcelain | grep -vE '^( D|D )' | grep .   # empty output = clean
```

## 3. Remove, in small batches of 3

Full checkouts (node_modules, .venv) are hundreds of thousands of files, so removal is heavy disk I/O. Go strictly sequential, 3 at a time, to keep load flat:

```bash
git worktree remove --force <path>   # --force needed because sparse phantoms read as dirty
git branch -D <branch>               # then delete its branch
```

For free branches not held by any worktree: `git branch -d` for merged, `git branch -D` for gone-upstream. Finish with `git worktree prune`.

If a removal fails with "Operation not permitted", the sandbox is blocking writes to `.git` and the worktree dir. Retry that command with the sandbox disabled (in Claude Code, `dangerouslyDisableSandbox: true`).

## 4. Optional second pass: unmerged worktrees (opt-in only)

Only when the user explicitly asks to also drop unmerged worktrees. Stricter gate, all three required:
- **Not live-locked** (stale locks OK, see step 1).
- **Clean working tree** (step 2).
- **Fully pushed** - the branch exists on the remote at the same or an older SHA, so a session can recreate the worktree later with nothing lost:

```bash
git rev-list --count @{upstream}..<branch>   # 0 = fully pushed; no upstream or >0 = KEEP
```

Never drop a worktree with unpushed commits or uncommitted work, and never one that is live-locked.
