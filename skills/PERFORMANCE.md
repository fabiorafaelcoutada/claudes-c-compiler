# Performance Optimization Skills

This document describes the key performance optimizations implemented in the CCC codebase, focusing on minimizing allocations and improving code generation speed.

## 1. Allocation Reduction (Static Strings)

**Problem:** Register names (e.g., `reg_to_32`) were frequently allocating `String`s, leading to 420 allocations per file in simple cases.

**Solution:** Use `Cow<'static, str>` or `&'static str` for constant string returns.

**Example:**
*Before:*
```rust
fn reg_to_32(r: Reg) -> String {
    match r {
        Reg::Rax => "eax".to_string(),
        // ...
    }
}
```

*After:*
```rust
use std::borrow::Cow;

fn reg_to_32(r: Reg) -> Cow<'static, str> {
    match r {
        Reg::Rax => Cow::Borrowed("eax"),
        // ...
        _ => Cow::Owned(format!("r{}d", r.index()))
    }
}
```

**Application:** Use `Cow` whenever returning string constants from enums or lookups.

## 2. Format Optimization (Hot Paths)

**Problem:** `format!` macro allocations were prevalent in hot code generation paths, causing unnecessary heap allocations.

**Solution:** Replace `format!` with `emit_fmt` or static strings where possible.

**Technique:**
*   **Static Tables:** For instruction mnemonics like `addw`, `subw`, use a static `&str` table instead of `format!("{}{}", base, suffix)`.
*   **Split Emission:** Instead of `emit(&format!("mov {}, {}", src, dst))`, use `emit_fmt` to construct output or split into multiple writes.
*   **Inline Formatting:** Use `write!` directly to a buffer rather than creating intermediate strings.

**Application:** Check critical loops in `codegen` and `assembler` for `format!` usage.

## 3. String Interning (Future Skill)

While not fully implemented, moving repeated identifiers to an interner (e.g., `SymbolTable`) reduces memory usage and speeds up comparisons (ptr eq vs string cmp).

**Current State:** CCC uses string interning for macro names and some identifiers, but not ubiquitously.
