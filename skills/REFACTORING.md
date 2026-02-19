# Refactoring Skills

This document outlines the refactoring techniques and patterns used in the CCC codebase, derived from actual cleanup tasks.

## 1. State Threading (Reducing Parameter Count)

**Problem:** Functions like `gvn_dfs` and `process_block` accumulated 10-14 parameters, making the code hard to read and modify.

**Solution:** Introduce a Context or State struct to bundle mutable state and common references.

**Example:**
*Before:*
```rust
fn gvn_dfs(
    curr_block: BlockId,
    predecessors: &HashMap<BlockId, Vec<BlockId>>,
    dom_children: &HashMap<BlockId, Vec<BlockId>>,
    values: &mut HashMap<ValueId, Value>,
    // ... 10 more params
)
```

*After:*
```rust
struct GvnState<'a> {
    values: &'a mut HashMap<ValueId, Value>,
    // ... all other state
}

impl<'a> GvnState<'a> {
    fn dfs(&mut self, curr_block: BlockId) { ... }
}
```

**Application:** Use this pattern whenever a function signature exceeds 6-7 parameters, especially if many are passed through recursively.

## 2. Parameter Objects (Grouping Arguments)

**Problem:** Parsing functions like `parse_declaration_rest` took 22 parameters, many of which were boolean flags or related attributes.

**Solution:** Group related arguments into a struct.

**Example:**
*Before:*
```rust
fn parse_declaration_rest(
    is_typedef: bool,
    is_static: bool,
    is_extern: bool,
    alignment: Option<u32>,
    // ... 18 more
)
```

*After:*
```rust
struct DeclAttributes {
    is_typedef: bool,
    storage_class: StorageClass, // merges static/extern/etc
    alignment: Option<u32>,
}

fn parse_declaration_rest(attrs: DeclAttributes, ...)
```

**Application:** Use this for configuration-heavy functions, particularly in the Frontend (parser/sema).

## 3. Common Code Extraction

**Problem:** The x86-64 (`src/backend/x86`) and i686 (`src/backend/i686`) backends shared substantial logic for instruction emission and operand formatting, leading to code duplication.

**Solution:** Create a shared module (e.g., `x86_common`) and move logic there.

**Techniques:**
*   **Shared Helper Functions:** `gcc_cc_to_x86` was extracted to a free function.
*   **Template Parsing:** `substitute_*_asm_operands` was 150 lines of duplicate code. It was extracted by parametrizing the operand emission callback.
*   **Unified Enums/Types:** Shared register mappings where possible.

**Application:** When implementing a new backend (e.g., a new 32-bit RISC architecture), look for commonalities with existing backends (e.g., RISC-V 64) and extract shared traits or helpers first.
