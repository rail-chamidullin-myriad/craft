---
name: worktree-cleanup
description: Remove every local git worktree and branch whose work already exists on the remote, in one pass, and keep everything else. Checks merged branches, remote branches, pull request heads, and squashed or rebased copies, plus uncommitted, ignored, and live work in worktrees. Shows the plan, waits for a go-ahead, saves a restore list, then deletes. Use when the user says "clean up worktrees", "clean up branches", "prune worktrees", "remove merged branches", "worktree cleanup", or asks to tidy a repo with many worktrees or branches.
---

# Worktree cleanup (craft)

Delete every worktree and local branch that is safe to remove, and nothing else. Work is safe when it exists on the remote. Run every check below in the first pass, so the user never has to ask whether more can go.

## 1. Refresh what the remote has

- Fetch with `--prune`, and find the default branch from `origin/HEAD`.
- Fetch pull request heads into a temporary ref namespace when the host publishes them. Delete that namespace at the end.

  ```bash
  git fetch --prune origin
  git fetch origin '+refs/pull/*/head:refs/craft-cleanup/pull/*'            # GitHub
  git fetch origin '+refs/merge-requests/*/head:refs/craft-cleanup/mr/*'    # GitLab
  ```

- A repo without a remote can only treat branches merged into the default branch as safe.

## 2. Classify each local branch

A branch is safe when the first test that passes says so:

1. **Its tip is on the remote.** The default branch, a remote branch, or a pull request head contains the tip.

   ```bash
   git for-each-ref --contains <branch> refs/remotes refs/craft-cleanup --count=1
   ```

2. **Its content is on the remote.** Squash merges and rebases leave a tip no remote ref contains. Compare each file the branch changes since its merge base with the default branch. The file is covered when a remote commit holds the same content, or holds every line the branch added and none it removed.

Everything else holds unique work and stays. Never delete the default branch or a branch a worktree has checked out, unless that worktree goes too.

## 3. Classify each worktree

A worktree is safe to remove when all of these hold:

- Its branch or detached commit is safe per step 2.
- It has no uncommitted or untracked changes. Sparse-checkout phantom deletions (` D` lines) do not count.
- It holds no ignored files of value. Dependency and build output, such as `node_modules/` or `.venv/`, does not count, and neither do copies identical to the main checkout's. A Terraform state file, an IDE config, or local notes do count.
- It is not in use. A lock whose process id is alive, or a commit or file change in the last hour, means another session may be working there. A lock whose process is dead does not count.

Keep the main checkout and any worktree outside the repo's worktree folder, such as another tool's checkout.

## 4. Show the plan and wait

Show counts, then each group with its reason: worktrees to remove, branches to delete by the test that cleared them, and what stays by why. Wait for the user's go-ahead.

Example:

```
Remove 9 worktrees: 5 merged, 4 fully pushed with open PRs.
Delete 41 branches: 12 merged, 20 on a remote branch, 6 match a PR head, 3 squash-merged.
Keep 7 worktrees: 3 live sessions, 2 uncommitted changes, 1 Terraform state, 1 active in the last hour.
Keep 18 branches: lines found nowhere on the remote.
```

## 5. Delete

1. Save each branch name and commit to `$(git rev-parse --git-common-dir)/craft-cleanup/YYYY-MM-DD.txt`. Any entry restores with `git branch <name> <sha>` until git collects it.
2. Re-check each worktree right before removing it. Another session may have started work since the audit.
3. Remove worktrees a few at a time, since large checkouts are heavy on disk.
4. Delete the branches, then run `git worktree prune` and delete the temporary refs.

## 6. Report

Report what went, what stayed by reason, and the restore file. Offer to show the unique lines in kept branches, so the user can judge whether they are abandoned drafts.
