# Tenants

1. It should be a package that I personally want to use and find useful
2. We should start each feature with UX in mind - what does the final API look like to the user? Then figure out feasibility and implementation details.
3. We should have an end goal in mind, but break down development into smaller, manageable features

# AI Usage

some ideas how to have ai help dev

use context7 to analyze targets and tarchetypes documentation - provide claude with enough context to answer questions about targets accurately.

# Implementation (brainstorm)

These are thoughts, maybe these ideas will change

How do we go from yaml file to test_targets?

1. declare tests -> 
2. READ -> 
3. Run tests

How to implement each step?

1. Declare tests

Option: use YAML based declaration

- pros: Easy to understand, easy to define, git tracked
- cons: 

2. Read tests

Option: yaml package to read spec

- pros: Easy to parse and integrates well with R functions (e.g. `!expr`)
- cons:

3. Compile & Run tests

How do we transform YAML into runnable tests?

**Decision: Option A (dynamic) by default, with optional file output**
- Default: `test_targets()` runs tests in-memory, no files written
- Advanced: `test_targets(write_tests = TRUE)` writes test files for debugging/CI integration
- Gives simplicity by default, power for users who need it

**Option A: Dynamic compilation (in-memory)**
- Dynamically generate test functions and run them directly
- No test files written to disk

Pros:
- Simpler implementation—no file I/O, no state to manage
- Tests are ephemeral—user's test suite stays clean (no auto-generated clutter in tests/testthat/)
- Faster iteration for users (write YAML, run `tarnation::test_targets()`, get results)
- Clear separation: user's hand-written tests vs tarnation's generated tests

Cons:
- Tests don't integrate with `devtools::test()` (user can't see tarnation tests alongside their regular testthat tests)
- Harder to debug—generated code isn't persisted, harder to inspect
- Tests don't persist in git, so no record of test history/changes
- Can't run tests in CI/CD pipeline easily (need to call `tarnation::test_targets()` explicitly, not just `devtools::test()`)

**Option B: Generate test files**
- Write generated test files to tests/testthat/ (e.g., test-tarnation-generated.R)
- Tests are discovered and run by testthat normally via `devtools::test()`

Pros:
- Tests integrate seamlessly with `devtools::test()` and CI/CD
- Generated test code is persisted and inspectable for debugging
- Tests are part of the git record
- User can see all tests (hand-written + generated) in one place via `devtools::test()`
- Natural integration with R package testing workflow

Cons:
- More complex implementation—need to write/manage files, handle cleanup, avoid conflicts
- Generated files in user's test directory (visual clutter, git diffs)
- Need to handle idempotency—what happens if user runs twice? Overwrite previous generated tests?
- Coupling—tarnation now owns files in user's test suite