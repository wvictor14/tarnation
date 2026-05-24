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

3. Run tests

Option: Run with testthat

- pros: See tests in devtools::test (is this a pro?)
- cons: complex? Not sure how to implement