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

## The "Why" Rule (CRITICAL)
If the change is non-trivial, add a body after a blank line. 
* The code diff already shows *how* the code changed. Your commit body MUST explain *why* the change was made.
* Explain the business logic, the bug's root cause, or the architectural decision.
* Answer: Why is this change necessary? What problem does it solve?

## Execution
1. Draft the commit message internally following the rules above.
2. Use the Bash tool to execute the commit using: `git commit -m "<your_message>"`
3. If the commit is successful, output a brief success message and show the final commit message you used.
