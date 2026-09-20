---
name: create-github-issue
version: 1.0.0
description: Investigate a problem first, then create a concise, well-researched GitHub issue on the specified repo. If the target repo is not specified, ask the user with a popup menu. Invoke with /create-github-issue <problem description>.
triggers:
  - create github issue
  - open an issue
  - 開 issue
  - 建 issue
---

# Create GitHub Issue

Create a GitHub issue that is short but genuinely useful: before writing anything, you MUST investigate the problem in the codebase, understand the root cause, and figure out the solution direction. The issue body should read like it was written by someone who already knows where the bug lives.

## Step 1 — Determine the target repo

Known repo mapping (owner `LeshLiao`):

| User says              | Repo                          | Local path                                        |
| ---------------------- | ----------------------------- | ------------------------------------------------- |
| PaletteWalliOS / iOS   | `LeshLiao/PaletteWalliOS`     | `~/project/PaletteWallFullStack/ios/PaletteWalliOS/` |
| PaletteWallAndroid / Android | `LeshLiao/PaletteWallAndroid` | `~/project/PaletteWallFullStack/android/PaletteWallAndroid/` |
| online-store / frontend / backend / 官網 | `LeshLiao/online-store` | `~/project/PaletteWallFullStack/online-store/` |

If the user did NOT clearly specify which repo, do not guess — use the **AskUserQuestion** tool with a single-select question listing:

1. `PaletteWalliOS` — iOS app (Swift/SwiftUI)
2. `PaletteWallAndroid` — Android app (Kotlin)
3. `online-store` — Frontend + Backend
4. `Current repo` — the repo of the current working directory (resolve it via `git remote get-url origin`)

(The user can also type any other `owner/repo` via the built-in Other option.)

## Step 2 — Investigate before writing (mandatory)

Do NOT create the issue from the user's description alone. First:

1. Read the relevant source code (use the local path above; for other repos use the current working directory, or `gh` / GitHub code search if no local checkout exists) to locate where the problem actually lives. Reference concrete files as `path/to/File.swift:line`.
2. Understand the root cause (or, for a feature request, the exact place and shape of the change).
3. Figure out a concrete solution direction — which files/functions to change and how.
4. If the repo has a `spec/` directory or docs, make sure the proposed solution doesn't contradict them.
5. Check for duplicates: `gh issue list --repo <owner/repo> --state open --search "<keywords>"`. If a duplicate exists, report it to the user instead of creating a new issue.

Note: the PaletteWallAndroid repo is read-only for code changes, but reading it to investigate is fine.

## Step 3 — Draft the issue (keep it simple)

Write the issue in **English**, short and precise. Template:

```markdown
## Problem
<1-3 sentences: symptom + root cause, with file:line references>

## Solution
<concise, concrete steps: which files/functions to change and how>

## Acceptance criteria
- [ ] <observable, testable outcome>
```

Title: one short imperative sentence (e.g. `Fix favorite list not refreshing after unfavorite`).

Rules:

- No filler, no restating the obvious. Every sentence must carry information found during Step 2.
- Root cause and solution must point at real code locations you verified, not guesses.
- Add labels only if an existing label clearly fits (`gh label list --repo ...` to check).

## Step 4 — Confirm, then create

Show the user the draft (title + body), then use **AskUserQuestion** to confirm:

1. `Create it` — proceed
2. `Let me adjust` — take the user's edits, update the draft, ask again

Only after confirmation:

```bash
gh issue create --repo <owner/repo> --title "<title>" --body "<body>"
```

Finally, reply to the user (in 台灣繁體中文) with the issue URL and a one-line summary of the root cause you found.
