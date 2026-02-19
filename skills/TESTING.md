# Testing Skills

This document outlines the testing strategy used in the CCC project, focusing on integration and regression testing.

## 1. Integration Tests (`tests/`)

**Structure:**
Each test is a directory in `tests/` containing:
*   `main.c`: The C source file to compile.
*   `expected.stdout`: The expected output of the compiled program.
*   `expected.ret`: The expected exit code (optional, defaults to 0).
*   `expected.skip.{arch}`: Marker file to skip test on specific architecture.

**Workflow:**
1.  Add a new test directory (e.g., `tests/my_test`).
2.  Write `main.c` with a minimal reproduction.
3.  Write `expected.stdout` with the expected output.
4.  Run `cargo test --release`.

**Skill:** `compile_test(test_path)`

## 2. Regression Testing (Projects)

**Purpose:** Ensure complex projects like PostgreSQL, FFmpeg, and SQLite continue to compile correctly.

**Workflow:**
1.  Clone the external project.
2.  Set `CC=/path/to/ccc`.
3.  Build and run the project's test suite.
4.  Update status in `projects/` or `ideas/new_projects.txt`.

**Example:**
*   PostgreSQL: 237 regression tests pass.
*   FFmpeg: 7331 checkasm tests pass.

**Skill:** `run_project_tests(project_path)`

## 3. Unit Tests (`#[test]`)

**Purpose:** Verify isolated components (passes, lexer, parsing logic).

**Location:** Inline within source files (`src/frontend/lexer/mod.rs`, `src/passes/gvn.rs`).

**Workflow:**
1.  Add `#[cfg(test)] mod tests { ... }` at the bottom of the file.
2.  Write tests using standard assertions (`assert_eq!`).
3.  Run `cargo test --lib`.

**Application:** Use unit tests for tricky algorithms (dominator tree, register allocation) or specific edge cases.
