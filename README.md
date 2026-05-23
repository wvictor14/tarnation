# tarnation

<!-- badges: start -->
[![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![CRAN status](https://www.r-pkg.org/badges/version/testtargets)](https://CRAN.R-project.org/package=testtargets)
<!-- badges: end -->

The goal of testtargets is to allow users to easily test their targets pipelines. It provides a set of tools to create test cases, run them, and check the results. Similar to other common data pipeline-testing approaches, like dbt and dagster, tarnation allows users to define tests for their data pipelines using a declarative yaml-based approach. 

tarnation provides a few built-in tests, but users can also write custom tests.

Under the hood tarnation uses testthat to run the tests, but it abstracts away the details of how to write tests and how to run them. This allows users to focus on writing tests for their data pipelines, rather than worrying about the mechanics of how to run them.

Concepts that this package leans on:

- declarative programming
- configuration as code
- infrastructure as code


## Installation

You can install the development version of testtargets like so:

``` r
# Install dev package
# install.packages("pak")
pak::pak("wvictor14/tarnation")
```

## Usage

### Step 1. Define tests for targets using a yaml file:

`_target_tests.yml`

```yaml
targets:
  - name: penguins
    columns:
      - name: species
        data_tests:
          - not_null
```

here, name refers to the name of the target, and columns refers to the columns within that target. The data_tests section allows you to specify the tests you want to run on each column.

### Step 2. Run the `tarnation::test_targets()` command

```r
tarnation::test_targets()

# Found 2 targets, 1 tests
# 17:31:05 |
# 17:31:05 | 1 of 1 START test penguins$species$data_tests$not_null..................... [RUN]
# 17:31:06 | 1 of 1 PASS penguins$species$data_tests$not_null........................... [PASS in 0.99s]
# 17:31:07 |
# 17:31:07 | Finished running 1 tests in 1.23s.

# Completed successfully

# Done. PASS=1 WARN=0 ERROR=0 SKIP=0 TOTAL=1
```

## `data_tests`

Out of the box, tarnation ships with some generic data tests already defined: `unique`, `not_missing`, `accepted_values`

## FAQ

### How do I test one target at a time?

```r
tarnation::test_targets(names = "penguins")
```

Can also use `tidyselect` select helpers

```r
tarnation::test_targets(names = contains('penguins'))
```

### One of my tests failed, how can I debug it?

TBD

### What data tests should I add to my project?

- `not_missing`
- `not_unique`

### When should I run my data tests?

Changes to code

Changes to source data

Production pipelines (test if your assumptions about source data are still valid)

### Can I store my data tests in a directory other than the `tests` directory in my project?

### How do I run data tests on just my sources?

[do we need this feature] Special select on sources? 

### Can I set test failure thresholds?

`error_if` `warn_if`
