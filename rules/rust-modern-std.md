---
name: rust-modern-std
description: "Rust 1.100 idioms: std replacements for cfg_if/lazy_static/once_cell/scopeguard, assert_matches!, strict_* arithmetic, and the deprecated std::i32::MAX / std::char modules."
# TTSR: rust-skills `rust-modern-features` (stable 1.86–1.100). Inherits the
# profile-wide `interruptMode: never`.
# Conditions: the crate macros/paths that std now covers, `assert!(matches!(`,
# `.checked_*(..).expect(`/`.unwrap()` overflow checks, and legacy
# primitive-module constants/functions (deprecated 1.97/1.99).
condition:
  - '\bcfg_if!|\bcfg_if::'
  - '\blazy_static!|\bonce_cell::(?:sync|unsync)::'
  - '\bscopeguard::'
  - 'assert!\([ \t]*matches!\('
  - '\.checked_(?:add|sub|mul|div|rem|neg|shl|shr|pow)\([^()\n]*\)\.(?:expect|unwrap)\('
  - '\bstd::(?:i8|i16|i32|i64|i128|isize|u8|u16|u32|u64|u128|usize|f32|f64)::[A-Z][A-Z0-9_]*\b'
  - '\bstd::char::(?:from_u32|from_digit|from_u32_unchecked|MAX|REPLACEMENT_CHARACTER|UNICODE_VERSION)\b'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: use the std/language feature (stable up to 1.100)

| Instead of | Write (since) |
|---|---|
| `cfg_if::cfg_if! { … }` | `cfg_select! { unix => { … } _ => { … } }` (1.95) |
| `lazy_static!`, `once_cell::sync::Lazy` / `OnceCell` | `std::sync::LazyLock` / `OnceLock` (1.80 / 1.70); `LazyCell` for single-thread |
| `scopeguard::guard(…)` / `defer!` | `core::mem::DropGuard::new(value, \|v\| …)`, `.dismiss()` to cancel (1.100) |
| `assert!(matches!(v, P))` | `use std::assert_matches;` then `assert_matches!(v, P)` (1.96, not in prelude) |
| `a.checked_add(b).expect("overflow")` | `a.strict_add(b)` and the `strict_*` family (1.91) |
| `std::i32::MAX`, `std::f64::EPSILON` | `i32::MAX`, `f64::EPSILON` (legacy modules deprecated 1.99) |
| `std::char::from_u32(x)` | `char::from_u32(x)` (deprecated 1.97) |

Also available now: let chains `if let Some(a) = x && let Ok(b) = f(a)` (1.88, edition 2024), `if let` match guards (1.95), `Vec::pop_if`, `extract_if`, `get_disjoint_mut`, `array_windows`, `Option::map_or_default`, `bool::toggle`, `str::trim_prefix`.

If the crate's `rust-version` is below the version shown, or the dependency is already used for something std doesn't cover, move on.
