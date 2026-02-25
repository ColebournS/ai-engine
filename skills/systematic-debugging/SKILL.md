---
name: systematic-debugging
description: Diagnoses bugs, test failures, and unexpected behavior through structured root cause analysis. Uses a four-phase process (reproduce, isolate, hypothesize, fix) instead of guessing. Use when the user encounters a bug, error, test failure, crash, regression, unexpected behavior, or asks to debug, investigate, diagnose, troubleshoot, fix, or figure out why something isn't working or broke.
---

# Systematic Debugging

Find the root cause before attempting fixes. Guessing wastes time and creates new bugs.

## Instructions

### Step 1: Gather Context

Before touching any code, understand the problem:

1. **Read error messages carefully** — stack traces, error codes, log output. Read them completely, don't skim.
2. **Reproduce the issue** — can you trigger it reliably? What are the exact steps? If it's intermittent, note the conditions.
3. **Check recent changes** — run `git diff` and `git log` to see what changed recently. New dependencies, config changes, and environment differences are common culprits.
4. **Understand the system** — search the codebase to understand how the failing component works. Read the relevant source files, not just the error location.

Don't propose any fix until you've completed this step.

### Step 2: Isolate the Failure

Narrow down where the problem actually is:

1. **Trace the data flow** — follow the input from entry point to the failure. At each layer, verify the data is what you expect.
2. **Find the boundary** — identify the last point where things are correct and the first point where they're wrong. The bug is between them.
3. **Check assumptions** — what does the code assume about its inputs, environment, or dependencies? Which assumption is violated?
4. **Find working examples** — search the codebase for similar code that works. Compare it to the broken code. List every difference.

For multi-component systems (API -> service -> database, CI -> build -> deploy), add diagnostic logging at each component boundary to determine which layer fails.

### Step 3: Form a Hypothesis

State your theory clearly before making changes:

- "I think X is the root cause because Y"
- Be specific — "the config is wrong" is not a hypothesis. "The database URL is missing the port because the env var isn't set in this environment" is.

Test the hypothesis with the **smallest possible change**:

- Change one variable at a time
- Don't bundle multiple fixes
- If the change doesn't fix it, **revert it** and form a new hypothesis

If you've tried 3+ hypotheses without success, stop and reassess:

- The problem may be architectural, not a simple bug
- Re-read the error from scratch with fresh eyes
- Consider whether your mental model of the system is wrong
- Discuss with the user before attempting more fixes

### Step 4: Fix and Verify

Once you've confirmed the root cause:

1. **Write a failing test** that reproduces the bug — use the `test-writing` skill for this. The test must fail before the fix and pass after.
2. **Implement the fix** — address the root cause, not the symptom. One change, not a bundle of "while I'm here" improvements.
3. **Verify the fix** — run the failing test (now passes), run all related tests (still pass), confirm no new warnings or errors.
4. **Check for related instances** — search the codebase for the same bug pattern elsewhere. If the root cause is a misunderstanding (wrong API usage, missing null check), the same mistake may exist in other places.
5. **Document the root cause** — briefly explain to the user what caused the bug and why, so the knowledge is captured. This prevents the same class of bug from recurring.

## Debugging Principles

- **Evidence over intuition** — verify every assumption with actual output, logs, or test results
- **One change at a time** — if you change multiple things, you can't know which one fixed it (or broke something else)
- **Read before you write** — understand the code thoroughly before modifying it. Search for callers, related modules, and tests.
- **Symptoms lie** — the error message tells you where the problem was detected, not where it was caused. Trace backward to the source.
- **Simplify to reproduce** — strip away complexity until you have the smallest reproduction case. Smaller reproductions are easier to diagnose.

## Red Flags — Stop and Reconsider

If you find yourself doing any of these, stop:

- Trying fixes without understanding the root cause
- Changing multiple things at once "to save time"
- Ignoring parts of a stack trace or error message
- Saying "this shouldn't matter but let me try changing it"
- Applying fixes from Stack Overflow or search results without understanding them
- Fixing the same bug for the third time

These indicate you're guessing, not debugging. Return to Step 1.

## Error Handling

If the issue can't be reproduced:
→ Gather more data. Check logs, add diagnostic output, ask the user for reproduction steps. Don't guess at a fix for a problem you can't see.

If the root cause is in a dependency or library you can't modify:
→ Document the bug, implement a workaround at the boundary, and add a test that will catch it if the dependency behavior changes.

If investigation reveals the issue is environmental (timing, infrastructure, third-party service):
→ Document what you investigated. Implement appropriate handling (retries, timeouts, circuit breakers) and add monitoring or logging for future diagnosis.

If the codebase has no tests for the affected area:
→ Write characterization tests that capture current behavior before making changes. Use the `test-writing` skill for this.
