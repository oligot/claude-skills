---
name: commit-message
description: Use when the user asks for a commit message, asks to "commit this", or wants to summarize a diff. Generates Conventional Commits format with a body explaining the "why", not the "how".
allowed-tools: Bash, Read
---

# Commit Message Generator
Generate a Conventional Commits-formatted commit message from the current staged changes.

## Execution Steps
1. Run `git diff --staged` to see the staged changes. 
2. If nothing is staged, run `git diff`, but warn the user the changes aren't staged yet.
3. Identify the primary change type: `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `perf`, `chore`.
4. Pick a scope from the changed files (e.g., the top-level package or feature folder). Omit if global.
5. Write the subject line: `type(scope): imperative verb + what changed`. Keep under 72 chars. No period at the end.

## Optional: Reference Footer (Jira / GitHub issue)
If the user supplies a ticket/issue reference (or the branch name or context clearly contains one), add it as a git trailer in the footer block. This **complements**, and does not replace, the `Co-Authored-By:` trailer.

* **Default to `Refs:`** — the neutral form that works for both Jira (`Refs: PROJ-123`) and GitHub (`Refs: #123`) with no side effects.
* **Use `Closes:` / `Fixes:` only** when the change fully resolves a GitHub issue (these auto-close it on merge to the default branch). Confirm with the user before using an auto-closing keyword.
* Skip this step entirely when no reference is available — do not invent one.
* Place the reference trailer **above** `Co-Authored-By:`, in the trailer block at the bottom, separated from the body by a blank line.
* **Keep all trailers contiguous** — no blank line *between* `Refs:` and `Co-Authored-By:`. Git only treats the last unbroken block of `Token: value` lines as trailers; a blank line between them drops the earlier ones. This means the commit must be made with a single message (e.g. `git commit -F <file>` or one `-m` containing newlines), **not** multiple `-m` flags (each `-m` inserts a blank line).

Example:
```
feat(auth): add SSO login

Enterprise customers need single sign-on to meet their
security compliance requirements.

Refs: PROJ-123
Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
```

## The "Why" Rule (CRITICAL)
If the change is non-trivial, add a body after a blank line. 
* The code diff already shows *how* the code changed. Your commit body MUST explain *why* the change was made.
* Explain the business logic, the bug's root cause, or the architectural decision.
* Answer: Why is this change necessary? What problem does it solve?

## Execution
1. Draft the commit message internally following the rules above.
2. Use the Bash tool to execute the commit using: `git commit -m "<your_message>"`
3. If the commit is successful, output a brief success message and show the final commit message you used.
