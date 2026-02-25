---
name: test-writing
description: Writes tests using test-driven development (RED-GREEN-REFACTOR). Discovers project test patterns, frameworks, and conventions through code search before writing any tests. Use when the user wants to write tests, add test coverage, do TDD, fix a bug with a regression test, add a regression test, or asks to test, verify, validate, or cover code.
---

# Test Writing

Write tests using RED-GREEN-REFACTOR. Discover the project's test framework, patterns, and conventions through code search — never assume a specific stack.

## Instructions

### Step 1: Discover Test Patterns

Before writing any test, search the codebase to understand how tests work here:

1. **Find the test framework**: Search for test config files (`pytest.ini`, `pyproject.toml`, `jest.config`, `vitest.config`, `tsconfig`, `.mocharc`, `conftest.py`, etc.)
2. **Find existing tests**: Search for test files to understand naming conventions (`test_*.py`, `*.test.ts`, `*.spec.js`, `*_test.go`, etc.)
3. **Read 2-3 nearby tests**: Find tests for code similar to what you're testing. Note:
   - Import patterns and test utilities
   - How fixtures, factories, or test data are set up
   - Assertion style (assert, expect, should)
   - How mocks/stubs are used (or avoided)
   - How the test runner is invoked (scripts, make targets, mise tasks, etc.)
4. **Find test helpers**: Search for shared fixtures, factories, builders, or test utilities the project provides
5. **Check for test commands**: Look at `package.json` scripts, `Makefile`, `mise` tasks, or CI config for how tests are run

Adapt to whatever you find. Do not impose a framework or pattern the project doesn't use.

### Step 2: Determine What to Test

Based on the user's request, identify test cases:

- **New feature** → Test the public API/behavior the feature exposes. Cover the happy path, edge cases, and error conditions.
- **Bug fix** → Write a test that reproduces the bug first. The test must fail before the fix and pass after.
- **Refactoring** → Verify existing behavior is preserved. Write characterization tests if none exist for the code being changed.
- **Coverage gaps** → Read the untested code and identify branches, error paths, and boundary conditions that lack tests.

Prioritize testing behavior over implementation. Tests should answer "what does this do?" not "how does this work internally?"

### Step 3: RED — Write Failing Test

Write one test at a time. Each test should:

- **Test one behavior** — if the test name contains "and", split it
- **Have a clear name** — describe the expected behavior, not the method name
- **Use real objects** where possible — mocks only when the real dependency is impractical (network, database, filesystem, time)
- **Follow project conventions** discovered in Step 1

Run the test. Verify it fails for the right reason:

- It should **fail** (assertion mismatch), not **error** (import missing, syntax broken)
- The failure message should describe the missing behavior
- If the test passes immediately, you're testing existing behavior — write a different test

### Step 4: GREEN — Write Minimal Code

Write the simplest code that makes the failing test pass:

- Don't add features the test doesn't require
- Don't refactor yet
- Don't optimize

Run the test again. Verify:

- The new test passes
- All existing tests still pass
- No warnings or errors in test output

If existing tests break, fix the regression before proceeding.

### Step 5: REFACTOR — Clean Up

With all tests green, improve the code:

- Remove duplication
- Improve naming
- Extract helpers or utilities
- Simplify complex expressions

Run all tests after each change to confirm nothing breaks. Don't add behavior during refactoring.

### Step 6: Repeat

Go back to Step 3 for the next behavior. Continue until all identified test cases from Step 2 are covered.

### Step 7: Verify Completeness

Before marking work done:

- [ ] Every new behavior has a test
- [ ] Each test was seen failing before the implementation
- [ ] Edge cases covered (empty inputs, nulls, boundary values, concurrent access)
- [ ] Error paths covered (invalid input, missing resources, permission failures)
- [ ] All tests pass with clean output
- [ ] Tests use project conventions discovered in Step 1

## When Tests Fail Unexpectedly

If a test fails and you don't understand why:

→ **Stop writing more tests.** Switch to the `systematic-debugging` skill to investigate. Don't guess at fixes — diagnose the root cause first, then return to the RED-GREEN-REFACTOR cycle.

If multiple tests start failing after a change:

→ Revert the change. Write a smaller, more focused test. Make a smaller change. If the failure pattern is unclear, use `systematic-debugging` to trace the root cause before proceeding.

## Test Quality Principles

- **Test behavior, not implementation** — tests that break when you refactor internals are brittle
- **One assertion per concept** — multiple assertions are fine if they verify one logical outcome
- **Tests are documentation** — a reader should understand the system's behavior from test names alone
- **Fast tests** — prefer unit tests over integration tests where possible. Slow tests get skipped.
- **Independent tests** — no test should depend on another test's side effects or execution order
- **Deterministic tests** — tests should produce the same result every time. Avoid relying on wall clock time, random data, or external services without proper isolation

## Error Handling

If no test framework is configured in the project:
→ Ask the user which framework to use. Don't install one without asking.

If you can't find any existing tests to reference:
→ Use general best practices for the language and note that you couldn't verify project test conventions.

If the code under test has no clear public API (tightly coupled, global state):
→ Test at the nearest clean boundary. Note the coupling as a concern and suggest refactoring if appropriate.

If running tests requires special setup (Docker, database, environment variables):
→ Search for setup scripts, docker-compose files, or CI config to understand the test environment. Ask the user if the setup is unclear.
