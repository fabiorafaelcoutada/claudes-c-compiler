# Code Quality Skills

This document describes the code hygiene and best practices enforced in the CCC codebase, derived from cleanup tasks like `projects/cleanup_code_quality.txt`.

## 1. Visibility (Minimizing API Surface)

**Problem:** Many types and functions were declared `pub` unnecessarily, exposing internal details and making refactoring harder.

**Solution:** Prefer `pub(crate)` over `pub` to restrict visibility to the crate, or use module-level visibility (`pub(super)`).

**Rule:**
*   Only items required by `main.rs` or integration tests should be `pub`.
*   Module internals should be `pub(crate)` or private.
*   Check `mod.rs` files to ensure they don't accidentally export everything.

**Example:**
*Before:* `pub struct CodegenOptions`
*After:* `pub(crate) struct CodegenOptions`

## 2. Explicit Imports (Avoiding Wildcards)

**Problem:** `use crate::ir::ir::*` imported dozens of names into the namespace, creating confusion and making dependency tracking hard.

**Solution:** Replace wildcard imports with explicit lists of used items.

**Rule:**
*   No `use ...::*` except for:
    *   Test modules (`use super::*;`)
    *   Tightly coupled submodules (e.g., `parser` importing `ast` definitions)
*   Group imports logically (std, external, crate).

**Example:**
*Before:* `use crate::ir::*`
*After:* `use crate::ir::{Value, ValueId, BlockId};`

## 3. Module Organization

**Problem:** Large files or deep nesting make navigation difficult.

**Solution:** Split large modules into logical submodules, but keep the public API clean via re-exports.

**Rule:**
*   **Driver**: `src/driver/mod.rs` handles CLI parsing and pipeline orchestration.
*   **Frontend**: `src/frontend/` splits into `lexer`, `parser`, `sema`, `preprocessor`.
*   **Backend**: Each arch (`x86`, `arm`, `riscv`) has its own `codegen`, `assembler`, `linker` submodules.

**Check:** Ensure `mod.rs` files clearly define the public interface of the directory.
