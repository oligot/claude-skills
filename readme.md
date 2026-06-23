# claude-skills

Personal [Claude Code](https://claude.ai/code) skills plugin — git workflow helpers that enforce clean commit history, meaningful commit messages, and isolated worktrees.

## Why

Claude's default commit behaviour produces uninformative messages and leaves messy branch history. These skills give Claude structured guidance: write messages that explain *why* a change was made, and land feature branches as a single coherent commit.

## Skills

### `commit-message`

Generates a [Conventional Commits](https://www.conventionalcommits.org/) message from the current staged diff.

- Picks the right type (`feat`, `fix`, `refactor`, `docs`, …) and an optional scope from the changed files.
- Adds a body when the change is non-trivial, explaining the *why* — not what the diff already shows.
- Commits immediately.

Trigger: ask Claude to "commit this", "write a commit message", or invoke `/commit-message`.

### `squash-to-main`

Squashes all commits on the current branch (since diverging from the base branch) into one, then invokes `commit-message` to generate the message.

- Auto-detects the base branch from the remote HEAD; falls back to `main`.
- Guards against no-op squashes (no commits ahead of base).
- Warns before rewriting history and asks for confirmation.
- Requires a force-push if the branch was already pushed.

Trigger: ask Claude to "squash this branch", "clean up my commits before merge", or invoke `/squash-to-main`.

### `create-worktree`

Creates a new git worktree as a sibling directory under the repository root, on a branch prefixed with the project's commit type (`feat/…`, `fix/…`, `docs/…`).

- Detects the repository root from the git common dir (bare-repo layout with side-by-side worktrees).
- Derives a kebab-case slug for the folder and a `type/slug` branch name from the task.
- Guards against existing branches/paths and branches from the repo's default branch.
- Avoids native worktree helpers that bury worktrees under `.claude/` and mangle branch names.

Trigger: ask Claude to "create a worktree", "start a branch in a worktree", or "spin up a worktree for X".

## Installation

**Prerequisites:** Claude Code with plugin support (`claude plugin` CLI available).

```bash
# 1. Register the marketplace
claude plugin marketplace add https://github.com/oligot/claude-skills

# 2. Install the plugin
claude plugin install user-skills@claude-skills

# 3. Verify
claude plugin list
```

### Dotfiles (re-registration on new machines)

Add to `extraKnownMarketplaces` in your Claude Code settings:

```json
"extraKnownMarketplaces": {
  "claude-skills": {
    "source": {
      "source": "url",
      "url": "https://github.com/oligot/claude-skills.git"
    }
  }
}
```

Then run `claude plugin install user-skills@claude-skills` on each new machine.

## Usage

All skills are invoked automatically when Claude detects the intent, or explicitly via slash commands:

```
/commit-message     — stage your changes first, then invoke
/squash-to-main     — run from the feature branch you want to squash
/create-worktree    — run from anywhere in the repo; describe the work to create the branch
```
