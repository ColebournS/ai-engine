---
name: code-review
description: Reviews code for bugs, security vulnerabilities, performance issues, edge cases, and consistency with codebase patterns. Use when the user wants a code review, audit, second opinion, double-check, quality check, sanity check, or asks to review, inspect, or check changes, diffs, files, or pull requests.
---

# Code Review

Perform a thorough code review acting as an experienced, critical reviewer. Focus on finding real issues — not style nitpicks — and always verify findings against the actual code before reporting them.

## Instructions

### Step 1: Determine Review Scope

Ask or infer what to review:

- **Git changes** → Use `git diff`, `git diff --staged`, or `git log` to get the changeset
- **Pull request** → Use `gh pr diff` or `gh pr view` to get PR context
- **Specific files/modules** → Read the files directly
- **Recent work** → Check git log for recent commits on the current branch

If the user hasn't specified, default to reviewing uncommitted changes (`git diff` + `git diff --staged`).

### Step 2: Gather Context

Before reviewing, understand the surrounding code:

1. Read the changed files in full (not just the diff hunks) to understand the broader context
2. Identify related files — callers, callees, tests, types, and interfaces that interact with the changed code
3. Check for project conventions — look at nearby files for patterns (naming, error handling, structure)
4. Read relevant tests to understand expected behavior

### Step 3: Review the Code

Analyze the changes through each of these lenses, in order of severity:

**Critical (must fix)**
- Logic errors and bugs — incorrect conditions, off-by-one errors, wrong variable used
- Security vulnerabilities — injection, auth bypass, secrets exposure, unsafe deserialization
- Data loss or corruption risks — missing transactions, race conditions, unchecked mutations
- Breaking changes to public APIs or contracts

**Important (should fix)**
- Missing error handling — unhandled exceptions, missing null checks, swallowed errors
- Edge cases not covered — empty inputs, boundary values, concurrent access
- Performance issues — N+1 queries, unnecessary allocations, missing indexes, blocking calls in async paths
- Resource leaks — unclosed connections, missing cleanup, unbounded growth

**Suggestions (consider)**
- Readability improvements — unclear naming, complex expressions that could be simplified
- Consistency with codebase patterns — deviations from established conventions
- Missing or misleading comments on non-obvious logic
- Test coverage gaps for new or changed behavior
- Opportunities to reduce duplication

### Step 4: Verify Findings

For every issue you identify:

1. Re-read the surrounding code to confirm the issue is real (not a misunderstanding)
2. Check if the issue is handled elsewhere (a guard clause upstream, a wrapper that catches errors, etc.)
3. Check if it's an existing pattern in the codebase (intentional, even if not ideal)
4. Only report issues you are confident about — false positives erode trust

### Step 5: Run Automated Checks

Before reporting, run available automated checks to corroborate findings:

1. Check for linter errors on changed files (use lint tools if available)
2. Run related tests if a test runner is configured — note any failures
3. Check for type errors if the project uses a type checker

Don't duplicate what automated tools catch — focus your review on issues that require human judgment. If automated checks surface issues, include them briefly under the appropriate severity.

### Step 6: Report Findings

Present the review in this format:

```
## Code Review Summary

**Scope**: [what was reviewed — files, diff range, PR number]
**Overall**: [one-sentence assessment — is this ready to merge, needs changes, or needs rework?]

### Critical Issues
- **[File:Line]** — [description of the bug/vulnerability]
  - Why: [brief explanation of the impact]
  - Fix: [concrete suggestion]

### Important Issues
- **[File:Line]** — [description]
  - Why: [impact]
  - Fix: [suggestion]

### Suggestions
- **[File:Line]** — [description and suggestion]

### What Looks Good
- [call out well-written code, good patterns, thorough tests — be specific]
```

If there are no issues at a severity level, omit that section. Always include "What Looks Good" — positive feedback matters.

## Review Principles

- **Be specific**: Point to exact lines and explain the concrete problem, not vague concerns
- **Propose fixes**: Every issue should include a suggested resolution
- **Assume competence**: The author likely had reasons for their choices — ask "why" before assuming it's wrong
- **Prioritize**: A review with 3 real issues is more valuable than one with 20 nitpicks
- **Verify before reporting**: Re-read the code to confirm every finding. If unsure, say so explicitly
- **Consider the diff, not just the code**: Focus more on what changed than pre-existing issues, unless asked for a full review

## Error Handling

If no changes are found (clean working tree, empty diff):
→ Tell the user there's nothing to review and ask what they'd like reviewed

If the codebase is unfamiliar (no clear patterns to reference):
→ Review against general best practices and note that you couldn't verify codebase conventions

If a file is too large to read fully:
→ Focus on the changed sections and their immediate context, noting that a full-file review wasn't performed

If you're uncertain about a finding:
→ Flag it with "[Uncertain]" and explain your reasoning — let the author decide
