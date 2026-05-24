# tarnation MVP Design

**Date:** 2026-05-24  
**Status:** Design (awaiting user review)

## Vision

tarnation is a declarative testing framework for targets pipelines. Users declare tests in YAML, the package compiles them into runnable tests, executes them, and reports which targets meet their specifications. This mirrors testing approaches in dbt and dagster.

## User Journey (Happy Path)

1. User has `_targets.R` with target definitions (e.g., targets::tar_target("penguins", ...))
2. User creates `_target_tests.yml` declaring tests for those targets
3. User calls `tarnation::test_targets()` 
4. Package discovers targets store (cache by default, or custom via parameter), loads target data
5. Package reads YAML, compiles into test functions, runs them via testthat
6. Package reports results: pass/fail, failed row counts, details available for inspection
7. User iterates: adjust tests or targets based on failures

**Example `_targets.R`:**
```r
library(targets)

tar_source()

list(
  tar_target(penguins, load_data("penguins.csv")),
  tar_target(summary_stats, summarize(penguins))
)
```

**Corresponding `_target_tests.yml`:**
```yaml
targets:
  - name: penguins
    columns:
      - name: species
        data_tests:
          - not_null
  - name: summary_stats
    columns:
      - name: count
        data_tests:
          - not_null
```

## Core Requirements

### Inputs

**YAML file structure** (`_target_tests.yml` or custom path):

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
  - name: summary_stats
    columns:
      - name: count
        data_tests:
          - not_null
```

**Built-in data tests (v1):**
- `not_null` — all values non-missing (no NA)
- `unique` — all values are distinct
- `accepted_values` — all values in allowed set (takes `values` param)

**Custom tests:**
- User can inline R expressions via YAML `!expr` tag (v1 feature, exact syntax TBD)
- Example: `!expr x > 0` evaluates as R code

### Processing

The package must:

1. **Discover targets** (in this order):
   - If user specifies `store` parameter, use that (e.g., `test_targets(store = "_targets/objects")`)
   - Otherwise, discover targets store in working directory (default: `_targets`)
   - Error: if store not found and not specified, fail with clear message ("No targets store found. Run `targets::tar_make()` first or specify `store = ...`")
2. **Load targets** from the store and get list of available target names
3. **Read** the YAML file from default location (`_target_tests.yml`) or user-specified path
4. **Parse** the YAML into an in-memory data structure (target → column → test declarations)
5. **Validate** that:
   - All referenced targets exist in the loaded targets store
   - Error: if target not found, fail with message ("Target 'unknown_target' not found. Available targets: penguins, summary_stats")
   - Warn: if YAML declares tests for zero targets
6. **Compile** test declarations into test functions
   - Each test declaration becomes a function that accepts a target's data, runs the check, returns pass/fail + details
   - Tests are generated dynamically at runtime (no files written by default)
7. **Execute** tests via testthat framework
8. **Collect** results including:
   - Which rows failed each test (if applicable)
   - Pass/fail status per test
   - Test execution time (optional)
   - Detailed results for inspection
9. **Report** results with:
   - Summary: PASS/FAIL counts
   - Per-test output showing failed row counts
   - Verbosity control: compact vs. detailed output

### Outputs

**User-facing (default behavior):**
```
Running tarnation tests...

penguins:
  ✓ species$not_null [PASS]
  ✓ island$accepted_values [PASS]
  ✗ island$not_null [FAIL] — 3 rows failed

summary_stats:
  ✓ count$not_null [PASS]
  ✗ count$unique [FAIL] — 5 duplicate values

Summary: 4 PASS, 2 FAIL
```

**For failures, additional details available** (inspection API TBD, e.g., `test_results$failures$penguins$island$not_null`):
- Row indices/values that failed
- Expected vs. actual values
- Configurable verbosity (quiet, normal, verbose)

## Architecture

### High-Level Data Flow

```
[discover_store] ← (default: _targets/ or custom via store param)
  ↓
[load_target_names] → List of available targets
  ↓
[read_yaml_tests] → Read _target_tests.yml (or custom path)
  ↓
[parse_tests] → Parsed test declarations (target/column/test structure)
  ↓
[validate_targets] → Check all declared targets exist in store
  ↓
[load_target_data] → Get target data from store for declared targets
  ↓
[compile_tests] → Test functions (closures capturing target data + test logic)
  ↓
[run_tests] → Results (pass/fail, failed rows, timing)
  ↓
[format_results] → Human-readable output
```

### Core Functions (Internal + Public)

**Public API:**
- `test_targets(path = "_target_tests.yml", store = NULL, verbosity = "normal", write_tests = FALSE)` — main entry point
  - `path` — custom YAML path (default: `_target_tests.yml`)
  - `store` — path to targets store (default: auto-discover `_targets/` in working directory)
  - `verbosity` — output detail level ("quiet", "normal", "verbose")
  - `write_tests` — (v1.1+) write generated test files for inspection/CI integration

**Internal functions:**
- `discover_store(store)` — find targets store (uses provided path or discovers default)
- `load_target_names(store)` — get list of target names in store
- `read_yaml_tests(path)` — read YAML file, return raw text/parsed list
- `parse_test_declarations(yaml_list)` — convert YAML structure into R data structure
- `validate_targets(declarations, target_names)` — check all referenced targets exist, error if mismatch
- `load_target_data(store, target_names)` — load target objects from store
- `build_test_functions(declarations, target_data)` — create test functions for each declaration
- `run_test_functions(test_fns)` — execute tests, collect results
- `format_test_results(results, verbosity)` — format for human consumption
- `inspect_failures(results, target, column)` — (API for users to dig into failures)

**Test implementations (built-in):**
- `test_not_null(data, column)` — check for NA values
- `test_unique(data, column)` — check for duplicates
- `test_accepted_values(data, column, values)` — check membership in set

### Data Structures (Conceptual)

**Parsed test declaration:**
```r
list(
  targets = list(
    list(
      name = "penguins",
      columns = list(
        list(
          name = "species",
          tests = list(
            list(test = "not_null"),
            list(test = "custom", expr = "!expr nchar(x) > 0")
          )
        ),
        list(
          name = "island",
          tests = list(
            list(test = "accepted_values", values = c("Biscoe", "Dream", "Torgersen"))
          )
        )
      )
    )
  )
)
```

**Test result:**
```r
list(
  target = "penguins",
  column = "species",
  test = "not_null",
  passed = TRUE,
  failed_rows = integer(0),
  failed_values = NA,
  elapsed_time = 0.001,
  message = "All 344 values are non-missing"
)
```

## Implementation Strategy

### Phase 1: Core Engine (Sprint 1-2)
- YAML reading (`read_yaml_tests`)
- Test declaration parsing (`parse_test_declarations`)
- Target validation (`validate_targets`)
- Built-in test implementations (not_null, unique, accepted_values)
- Test function compilation (`build_test_functions`)
- Test execution (`run_test_functions`)
- Basic result formatting

### Phase 2: Reporting & UX (Sprint 3)
- Result formatting with verbosity control
- Failure inspection API
- Human-readable output

### Phase 3+: Advanced Features (Later)
- Custom tests via !expr (v1 or v1.1)
- Write test files for CI integration (v1.1)
- Filtering/selection helpers (v2)
- Error thresholds (v2)
- Debugging utilities (v2)

## Dependencies

- `yaml` — parse YAML files
- `testthat` — (suggested) for testing framework; used internally for test infrastructure
- `targets` — (suggested) to load and interact with target pipeline

No other hard dependencies; monitor for bloat.

## Success Criteria

**MVP is complete when:**
1. User can declare tests in YAML with targets, columns, data_tests
2. `test_targets()` reads YAML, runs tests, reports pass/fail
3. Built-in tests (not_null, unique, accepted_values) work correctly
4. Failures show which rows failed
5. Results are stored for inspection
6. Verbosity control works
7. All functions tested with real/realistic examples
8. Code formatted per project standards (`air format .`)

## Open Questions / TBD

1. **Custom test syntax (v1 or v1.1?):** Exact YAML syntax for `!expr` or separate function file approach
2. **Failure inspection API:** What function(s) let users dig into failure details? (e.g., `results$failures`, `inspect_failures(results, "penguins", "species")`)
3. **Partial failures:** If one column's test fails, do we continue running other tests, or stop? (Current assumption: continue)
4. **Row-level vs. count-level reporting:** For `unique` failures, show each duplicate, or just count?
5. **Empty test file:** If `_target_tests.yml` exists but declares zero targets/tests, should we warn or fail?
6. **No tests for a target:** If a target exists but no tests declared for it, is that OK or warn?

## Notes

- Aligns with tarnation tenants: user-first design (UX first), incremental development (phased approach), personally useful (declarative > imperative)
- Keeps dependencies lightweight (yaml + existing infrastructure)
- Default dynamic compilation (no file clutter); opt-in file writing for power users
