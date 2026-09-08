---
name: git-commit-one-line
version: 1.0.0
description: Read git diff and commit with a concise one-line English message.
triggers:
  - git commit
  - commit changes
  - commit this
allowed-tools:
  - Bash
---

## When to invoke this skill

Use when the user wants to commit current changes with an auto-generated message.
Triggers: "git commit", "commit changes", "commit this", "幫我 commit", "commit 一下".

## Steps

1. Run `git diff HEAD` to read all staged and unstaged changes.
2. Also run `git status` to see which files are affected.
3. Based on the diff, write a single concise commit message in English:
   - One line only, no bullet points, no body
   - Start with a dash and a space, then a verb: `- Add / - Fix / - Update / - Remove / - Refactor / etc.`
   - Focus on WHAT changed and WHY (if obvious from diff)
   - Max ~72 characters
4. Stage all files (including untracked) with `git add -A`, then commit with the message using `git commit -m "..."`.
5. Show the final commit hash and message to the user.

## Rules

- Message must be in English, one line only.
- Use `git add -A` to include untracked files, then `git commit -m "..."`.
- Do NOT amend existing commits.
- Do NOT skip hooks (no `--no-verify`).
- If there is nothing to commit, say so and stop.

## Example messages

- `- Add drag-to-reorder support for sticker layer`
- `- Fix animation not resetting on screen-off`
- `- Update save flow to use synchronous SharedPrefs commit`
- `- Remove unused legacy template fields`
