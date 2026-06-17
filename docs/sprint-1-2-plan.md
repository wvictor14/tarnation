# Sprint 1-2 implementation plan

**Date:** 2026-06-17  
**Scope:** Phase 1 of MVP — core engine  
**Spec:** [2026-05-24-tarnation-mvp-design.md](superpowers/specs/2026-05-24-tarnation-mvp-design.md)

## Decisions (deviations from spec)

| Topic | Spec | Decision |
|---|---|---|
| Package name | `testtargets` | `tarnation` — fix DESCRIPTION |
| `load_target_names` | Internal wrapper | Call `targets::tar_objects(store)` inline |
| `load_target_data` | Internal wrapper | Call `targets::tar_read(name, store)` inline |
| `targets` dependency | Suggests | Imports — required at runtime |
| Test execution | "Via testthat" | Plain R result lists — no `expect_*` in execution path |
| Failure messages | Not specified | Use `waldo` for rich diffs (Phase 2) |
| `build_test_functions` | Separate compile step | Drop — dispatch directly to test implementations |
| Store discovery default | `_targets/` | `targets::tar_config_get("store")` — respects project config |

## Dependencies

Add to `DESCRIPTION`:

```
Imports:
  targets,
  yaml,
  cli
Suggests:
  testthat (>= 3.0.0),
  waldo
```

`waldo` deferred to Phase 2 (reporting UX). `cli` needed for formatted output from day one.

## File structure

```
R/
  test_targets.R     # public API: test_targets()
  parse.R            # read_yaml_tests(), parse_test_declarations()
  validate.R         # validate_targets()
  run.R              # run_tests(), dispatch_test()
  tests.R            # test_not_null(), test_unique(), test_accepted_values()
  results.R          # format_test_results()
tests/testthat/
  test-parse.R
  test-validate.R
  test-run.R
  test-tests.R
  fixtures/
    simple.yml         # minimal YAML fixture for unit tests
    _targets/          # minimal targets store fixture for integration tests
```

## Sprint 1: parsing and validation (no test execution)

Goal: given a YAML file and a store, we can parse and validate — all with unit tests, no real targets store needed yet.

**Setup**
- [ ] Fix `DESCRIPTION`: package name → `tarnation`, add Imports (`targets`, `yaml`, `cli`), update URL/BugReports
- [ ] `_pkgdown.yml`: scaffold reference index for `test_targets()`

**`R/parse.R`**
- [ ] `read_yaml_tests(path)` — calls `yaml::read_yaml(path)`, errors with `cli::cli_abort` if file not found
- [ ] `parse_test_declarations(yaml_list)` — converts raw YAML list into canonical structure (see spec data structure)
- [ ] Tests: parse well-formed YAML, missing `targets` key, empty targets list, unknown test type

**`R/validate.R`**
- [ ] `validate_targets(declarations, store)` — calls `targets::tar_objects(store)`, errors if any declared target is missing with list of available targets
- [ ] Tests: all targets present, one missing, all missing

**Fixture: `tests/testthat/fixtures/simple.yml`**
```yaml
targets:
  - name: penguins
    columns:
      - name: species
        data_tests:
          - not_null
      - name: island
        data_tests:
          - accepted_values:
              values: ["Biscoe", "Dream", "Torgersen"]
```

## Sprint 2: test execution and results

Goal: end-to-end `test_targets()` works against a real targets store.

**`R/tests.R`** — built-in test implementations
- [ ] `test_not_null(data, column)` → result list
- [ ] `test_unique(data, column)` → result list
- [ ] `test_accepted_values(data, column, values)` → result list
- [ ] Each returns: `list(passed, failed_rows, n_total, message)`
- [ ] Tests: passing case, failing case with correct `failed_rows`, edge cases (all NA, zero rows, empty column)

**`R/run.R`**
- [ ] `dispatch_test(data, column, test)` — routes to the right `test_*` function by `test$type`
- [ ] `run_tests(declarations, store)` — iterates declarations, calls `targets::tar_read()` per target, dispatches tests, returns flat list of results
- [ ] Tests: full run against fixture store, unknown test type errors clearly

**`R/results.R`**
- [ ] `format_test_results(results, verbosity)` — basic pass/fail output matching spec's format
- [ ] Invisibly returns results list (for programmatic inspection)
- [ ] Tests: PASS output, FAIL output with row count, summary line

**`R/test_targets.R`** — public API
- [ ] `test_targets(path, store, verbosity)` — orchestrates the full pipeline
- [ ] Store defaulting: `store %||% targets::tar_config_get("store")`
- [ ] Export and roxygen2 docs, add to `_pkgdown.yml`

**Fixture: `tests/testthat/fixtures/_targets/`**
- [ ] Minimal targets store with `penguins` target (a small data frame) for integration tests
- [ ] Use `targets::tar_make()` or manually create the store objects

## Result data structure (canonical)

```r
list(
  target  = "penguins",
  column  = "species",
  test    = "not_null",
  passed  = FALSE,
  failed_rows   = c(6L, 11L),
  n_total = 344L,
  message = "6/344 values are NA"
)
```

## Out of scope for Sprint 1-2

- `waldo` diffs in failure messages (Phase 2)
- Verbosity control beyond basic pass/fail (Phase 2)
- `inspect_failures()` API (Phase 2)
- Custom tests via `!expr` (Phase 3)
- `write_tests = TRUE` (Phase 3)
