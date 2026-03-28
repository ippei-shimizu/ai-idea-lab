---
name: auto-commit-push
description: >
  Automatically commit all changes and push to the current remote branch without user approval prompts.
  Use when the user says "コミットしてプッシュして", "commit and push", "auto commit", "変更をプッシュ",
  or any request to commit and push changes together. Do NOT use for commit-only or push-only requests.
---

# Auto Commit & Push

Automatically stage, commit, and push all changes to the current branch.

## Workflow

1. Run `git status` and `git diff` (staged + unstaged) in parallel to understand changes.
2. Run `git log --oneline -5` to match the repo's commit message style.
3. Stage all changed/new files with `git add` (list specific files, avoid `git add -A`). Skip `.env`, credentials, and secret files.
4. Generate a concise commit message in the same language and style as recent commits. Focus on "why" not "what". Do NOT add a Co-Authored-By line.
5. Commit using a HEREDOC for the message.
6. Push to the current branch's remote tracking branch. If no upstream is set, use `git push -u origin <current-branch>`.
7. Report the commit hash and push result to the user.

## Rules

- Never prompt the user for approval — execute all steps automatically.
- If there are no changes, inform the user and stop.
- If pre-commit hooks fail, fix the issue automatically if possible, then retry with a NEW commit (never amend).
- Do not include `.env`, credentials, or secret files in commits. Warn the user if such files are detected.
