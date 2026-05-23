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

## Example

This is a basic example which shows you how to solve a common problem:

```r
library(testtargets)
## basic example code
```

