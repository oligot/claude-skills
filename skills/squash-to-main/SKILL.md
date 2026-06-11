---
name: squash-to-main
description: Use when the user asks to squash commits, clean up branch history before merging, or says "squash this branch" / "squash my commits" / "clean up my commits before merge". Squashes all branch commits since the base branch into one and generates a conventional commit message.
allowed-tools: Bash, Skill
---

# Squash Branch to Single Commit

Squash all commits on the current branch (since diverging from the base branch) into a single commit, then invoke the `commit-message` skill to generate the message.

## Steps

1. **Auto-detect the base branch** (no network call):
   ```bash
   git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'
   ```
   If the command fails or returns empty, fall back to `main`. Use this detected branch name in place of `<base-branch>` throughout the remaining steps.

2. **Capture the divergence point:**
   ```bash
   MERGE_BASE=$(git merge-base HEAD <base-branch>)
   ```

3. **Guard — count commits ahead of base:**
   ```bash
   git rev-list --count $MERGE_BASE..HEAD
   ```
   - If the count is 0, stop and tell the user: "No commits to squash — this branch has no commits ahead of `<base-branch>`."
   - If the count is 1, inform the user: "There is only one commit to squash — this will reword the commit message without changing history structure." Then proceed.

4. **Warn the user and ask for confirmation** before proceeding:
   - This rewrites history. The original commit hashes will be lost.
   - If the branch has already been pushed to a remote, a force-push will be required after squashing.
   - **Ask the user to confirm before continuing.**

5. **Soft-reset to the divergence point:**
   ```bash
   git reset --soft $MERGE_BASE
   ```
   This stages all changes from the squashed commits without touching the working tree.

6. **Invoke the `commit-message` skill** — it sees the full staged diff and generates a conventional commit message, then commits.
