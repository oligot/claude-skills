---
name: create-worktree
description: Use when the user wants to start work in a new git worktree, asks to "create a worktree", "start a branch in a worktree", "spin up a worktree for X", or begin an isolated feature/fix/docs change. Creates the worktree as a sibling under the repository root with a branch prefixed by the project's commit type (feat/…, fix/…, docs/…).
allowed-tools: Bash
---

# Create Git Worktree

Create a new git worktree as a sibling directory under the repository root, on a new branch whose name is prefixed with the same type used for [Conventional Commits](https://www.conventionalcommits.org/) — `type/description`.

This assumes the **bare-repo layout**: a `.bare` (or `.git`) common dir whose parent directory holds the worktrees side by side (e.g. `main/`, `feat-x/`). The new worktree goes next to the existing ones — never nested inside another worktree or under `.claude/`.

## Steps

1. **Detect the repository root** (the parent of the git common dir):
   ```bash
   ROOT=$(dirname "$(git rev-parse --path-format=absolute --git-common-dir)")
   ```
   This resolves to the directory that contains the sibling worktrees (e.g. `/path/to/project`).

2. **Choose the `type`** from the nature of the work, using the same Conventional Commits types as the project's commit messages:
   `feat`, `fix`, `docs`, `refactor`, `chore`, `test`, `perf`.
   - New capability → `feat`; bug fix → `fix`; documentation → `docs`; etc.
   - If the intent is ambiguous, ask the user which type to use.

3. **Derive a kebab-case `SLUG`** (lowercase, hyphen-separated, no spaces or slashes) summarising the task — e.g. `ensure-secure-file-dest-dir`. The branch is `type/SLUG`; the directory is just `SLUG` (matching the existing sibling worktrees, whose folder names omit the `type/` prefix).

   **Strip a redundant type prefix.** If the slug you derived begins with one of the type words above followed by a hyphen, that leading word *is* the type — promote it to `TYPE` and drop it from `SLUG` so the type never appears twice. This turns a would-be `feat/feat-x` into `feat/x`, and `feat/fix-procedure` into `fix/procedure`:
   ```bash
   case "$SLUG" in
     feat-*|fix-*|docs-*|refactor-*|chore-*|test-*|perf-*)
       TYPE="${SLUG%%-*}"   # leading word becomes the type
       SLUG="${SLUG#*-}"    # drop the redundant "type-" prefix
       ;;
   esac
   ```

4. **Guard against collisions** before creating anything. If this prints anything, **STOP** — ask the user for a different slug or whether to reuse the existing worktree. Do not proceed to step 6:
   ```bash
   git show-ref --verify --quiet "refs/heads/$TYPE/$SLUG" && echo "ABORT: branch $TYPE/$SLUG already exists"
   [ -e "$ROOT/$SLUG" ] && echo "ABORT: path $ROOT/$SLUG already exists"
   ```
   (`git worktree add` would fail on either collision anyway; this check makes the reason explicit first.)

5. **Pick the base branch** (default branch of the repo, no network call):
   ```bash
   BASE=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||')
   BASE=${BASE:-main}
   ```

6. **Create the worktree on a new branch.** Prefer [workmux](https://workmux.raine.dev): it also opens a tmux window for the worktree and registers it for agent status tracking. Its global config (`worktree_dir: ..`, `worktree_naming: basename`) yields exactly `$ROOT/$SLUG`. Fall back to plain git when workmux is absent:
   ```bash
   # workmux must run from inside a worktree: it rejects the bare-repo root
   # ($ROOT itself, whose .git points at .bare and has no commondir).
   WM_DIR=$PWD
   [ "$(git rev-parse --is-inside-work-tree 2>/dev/null)" = true ] || WM_DIR="$ROOT/main"
   if command -v workmux >/dev/null 2>&1 && [ -d "$WM_DIR" ]; then
     # One tmux session per project, named after $ROOT (same rule as
     # tmux-sessionizer: "." -> "_"). Without --parent-session workmux puts the
     # window in whatever session this shell runs in, so a plan touching three
     # repos piles all its windows into one session.
     SESSION=$(basename "$ROOT" | tr . _)
     tmux has-session -t "=$SESSION" 2>/dev/null || tmux new-session -ds "$SESSION" -c "$ROOT"
     (cd "$WM_DIR" && workmux add "$TYPE/$SLUG" --base "$BASE" -b --parent-session "$SESSION")
   else
     git worktree add "$ROOT/$SLUG" -b "$TYPE/$SLUG" "$BASE"
   fi
   ```
   `-b` creates the window in the background so the current pane keeps focus. `--parent-session` targets the project's session rather than the current one; the window is created there even when you are working from another project's session.

7. **Report** the created path and branch, then run all subsequent work against `"$ROOT/$SLUG"` (use `git -C "$ROOT/$SLUG" …` or treat it as the new working directory). Confirm with `git worktree list`.

## Notes

- Branch the worktree from `$BASE` to start from the project's mainline. To start from current local HEAD instead, replace `"$BASE"` with `HEAD`; to start from the freshest remote state, `git fetch` first and use `"origin/$BASE"`.
- Do **not** use a native `EnterWorktree`/`--worktree` helper here — those place worktrees under `.claude/worktrees/` and sanitise `/` out of branch names, which breaks both the root-path convention and the `type/SLUG` branch name.
- To remove a worktree later: `workmux remove "$SLUG"` if it was created by workmux (also closes its tmux window), otherwise `git worktree remove "$ROOT/$SLUG"`. Both keep the branch. Use `git worktree move <old> <new>` to relocate one without losing commits — never delete the branch to "move" it.
