---
name: git-workflow
description: Manages git operations including commit messages, branch management, and pull request preparation. Discovers project git conventions through repo history. Use when the user wants to commit, create a branch, prepare a PR, write a commit message, clean up git history, push changes, or manage branches.
---

# Git Workflow

Handle git operations — commits, branches, and PR preparation — by discovering the project's conventions from its git history.

## Instructions

### Step 1: Discover Project Conventions

Before any git operation, learn how this project works:

1. **Read recent commits**: Run `git log --oneline -20` to see the commit message style. Note:
   - Format (conventional commits, free-form, ticket prefixes, etc.)
   - Tense (imperative "Add feature" vs past "Added feature")
   - Scope conventions (e.g., `feat(auth):` vs `fix: auth -`)
   - Whether commits reference issue/ticket numbers
2. **Check for commit hooks**: Look for `.husky/`, `.pre-commit-config.yaml`, or `git hooks` that enforce conventions
3. **Check branch naming**: Run `git branch -a` to see naming patterns (e.g., `feature/`, `fix/`, `chore/`, ticket-prefixed)
4. **Check for PR templates**: Look for `.github/pull_request_template.md` or similar

Follow whatever conventions you find. Don't impose a style the project doesn't use.

### Step 2: Determine the Operation

Based on the user's request:

- **"Commit this"** → Go to [Committing Changes](#committing-changes)
- **"Create a branch"** → Go to [Branch Management](#branch-management)
- **"Prepare a PR"** → Go to [Pull Request Preparation](#pull-request-preparation)
- **"Clean up commits"** → Go to [History Cleanup](#history-cleanup)

---

## Committing Changes

### Assess What Changed

1. Run `git status` to see all modified, staged, and untracked files
2. Run `git diff` and `git diff --staged` to understand the actual changes
3. Group related changes — if the diff contains unrelated work, suggest splitting into multiple commits

### Write the Commit Message

Based on the conventions discovered in Step 1:

- **Subject line**: Summarize the "why" not the "what". Keep it under 72 characters.
- **Body** (if needed): Explain motivation, trade-offs, or anything non-obvious. Wrap at 72 characters.
- **References**: Include ticket/issue numbers if the project convention expects them.

Stage the appropriate files and commit. Don't stage files that aren't related to the commit's purpose (e.g., unrelated formatting changes, lock files that weren't intentionally modified).

### Safety Rules

- Never use `--no-verify` to skip hooks unless the user explicitly requests it
- Never use `--force` push unless the user explicitly requests it and understands the consequences
- Never commit secrets, credentials, `.env` files, or API keys — warn the user if these are staged
- Don't amend commits that have been pushed to a remote unless the user explicitly requests it

---

## Branch Management

### Creating Branches

1. Match the project's branch naming convention discovered in Step 1
2. Create from the appropriate base branch (usually `main` or `develop` — check `git branch -a`)
3. If the user provides a ticket number or feature name, incorporate it into the branch name following project conventions

### Switching and Updating

- Before switching branches, check for uncommitted changes with `git status`
- If there are uncommitted changes, ask the user whether to stash, commit, or discard them
- After switching, suggest pulling latest if the branch is behind the remote

---

## Pull Request Preparation

### Step 1: Review Your Changes

1. Run `git log main..HEAD --oneline` (or the appropriate base branch) to see all commits that will be in the PR
2. Run `git diff main...HEAD` to see the full diff
3. Verify all changes are intentional — no debug code, no unrelated modifications, no commented-out blocks

### Step 2: Write the PR Description

Search for a PR template (`.github/pull_request_template.md`). If one exists, fill it in. If not, use this structure:

```
## Summary
[1-3 sentences: what this PR does and why]

## Changes
- [Key change 1]
- [Key change 2]

## Testing
- [How this was tested — automated tests, manual verification, etc.]
```

Keep it concise. Reviewers read dozens of PRs — respect their time.

### Step 3: Push and Create

1. Push the branch: `git push -u origin HEAD`
2. Create the PR using `gh pr create` with the title and description
3. Return the PR URL to the user

---

## History Cleanup

When the user asks to clean up commits before merging:

1. **Squash fixup commits**: Identify commits that are clearly fixes to earlier commits ("fix typo", "oops", "WIP")
2. **Reorder for clarity**: Group related changes so the commit history tells a coherent story
3. **Rewrite messages**: Ensure each commit message follows project conventions and explains the "why"

Use `git rebase` for cleanup. Never rebase commits that have been shared with others unless the user explicitly confirms it's safe.

## Error Handling

If `git push` is rejected (non-fast-forward):
→ Run `git pull --rebase` to incorporate remote changes. If there are conflicts, help the user resolve them before retrying.

If a merge conflict occurs:
→ Show the conflicting files and explain what each side changed. Help the user resolve conflicts, then verify the resolution doesn't break tests.

If the user has accidentally committed sensitive data:
→ Warn them immediately. Help remove it from history with `git filter-branch` or `git-filter-repo` if it hasn't been pushed, or advise rotating credentials if it has.

If the working tree is in a messy state (detached HEAD, mid-rebase, mid-merge):
→ Run `git status` to diagnose the state. Explain what happened and offer to abort or continue the in-progress operation.
