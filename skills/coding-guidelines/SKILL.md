---
name: coding-guidelines
description: "Use when asking about Rust code style or best practices. Keywords: naming, formatting, comment, clippy, rustfmt, lint, code style, best practice, P.NAM, G.FMT, code review, Google Rust style, google style guide, non_exhaustive, must_use, glob import, getter setter, naming convention, variable naming, function naming, type naming, 命名规范, 代码风格, 格式化, 最佳实践, 代码审查, 怎么命名"
source: https://rust-coding-guidelines.github.io/rust-coding-guidelines-zh/ + https://google.github.io/styleguide/rustguide.html
user-invocable: false
---

# Rust Coding Guidelines (50 Core Rules + Google Rust Style)

> Two layers: the core rules below, plus the **Google Rust Style Guide** (`google-style.md`, IDs `G.*`).
> When they conflict, Google style wins for new code. Rules tagged [chrome]/[Fuchsia] in that file are NOT defaults.

## Google Style — Top Rules (full list: `google-style.md`)

| ID | Rule |
|----|------|
| G.ORG.3 | No glob imports (except `use super::*` in tests, `use Enum::*` in fns about that enum, `pub use sub::*` re-exports) |
| G.ORG.4/6/7 | One module per `use` (no nested braces); call free fns as `mem::swap(..)`; use types unqualified (`Regex`) |
| G.OWN.1/2 | Avoid `Rc`/`Arc` and deep `.clone()` unless borrowing gets unreasonably complex |
| G.API.1 | `&str`/`&[T]`/`&T`/`Option<&T>` params, never `&String`/`&Vec<T>`/`&Box<T>`/`&Option<T>` |
| G.API.5 | `enum` instead of non-obvious `bool` params; otherwise `f(/* retry= */ true)` |
| G.TRT.4 | Don't define traits just to organize code — inherent `impl` unless needed for bounds/`dyn`/extension |
| G.NAM.4/6 | `new`/`new_with_x`/`try_new`; accessors `foo`/`foo_mut`/`set_foo`/`take_foo`/`replace_foo`/`with_foo` |
| G.FEA.2 | No nightly features in production |
| G.FEA.7/8 | `#[non_exhaustive]` on public enums/structs crossing team boundaries; `#[must_use]` where ignoring is a bug |
| G.IMP.1 | Annotate `let` types after `.collect()`/`.parse()`/`.into()` instead of turbofish |
| G.IMP.5 | Assertions carry context: `assert!(x > 0, "x must be positive, got {x}")` |
| G.IMP.6 | Prefer `for` loops over long combinator chains / `for_each` with side effects |
| G.IMP.7 | Never `catch_unwind` as control flow |
| G.MAC | functions → `macro_rules!` → codegen → proc-macro (in that order) |

## Naming (Rust-Specific)

| Rule | Guideline |
|------|-----------|
| No `get_` prefix | `fn name()` not `fn get_name()` |
| Iterator convention | `iter()` / `iter_mut()` / `into_iter()` |
| Conversion naming | `as_` (cheap &), `to_` (expensive), `into_` (ownership) |
| Statics / consts | `SCREAMING_SNAKE_CASE`, no prefixes (Rust API Guidelines) |

## Data Types

| Rule | Guideline |
|------|-----------|
| Use newtypes | `struct Email(String)` for domain semantics |
| Prefer slice patterns | `if let [first, .., last] = slice` |
| Pre-allocate | `Vec::with_capacity()`, `String::with_capacity()` |
| Avoid Vec abuse | Use arrays for fixed sizes |

## Strings

| Rule | Guideline |
|------|-----------|
| Prefer bytes | `s.bytes()` over `s.chars()` when ASCII |
| Use `Cow<str>` | When might modify borrowed data |
| Use `format!` | Over string concatenation with `+` |
| Avoid nested iteration | `contains()` on string is O(n*m) |

## Error Handling

| Rule | Guideline |
|------|-----------|
| Use `?` propagation | Not `try!()` macro |
| `expect()` over `unwrap()` | When value guaranteed |
| Assertions for invariants | `assert!` at function entry |

## Memory

| Rule | Guideline |
|------|-----------|
| Meaningful lifetimes | `'src`, `'ctx`; `'_` where elision allowed; `'a` only for local in→out relations (G.NAM.2) |
| `try_borrow()` for RefCell | Avoid panic |
| Shadowing for transformation | `let x = x.parse()?` |

## Concurrency

| Rule | Guideline |
|------|-----------|
| Identify lock ordering | Prevent deadlocks |
| Atomics for primitives | Not Mutex for bool/usize |
| Choose memory order carefully | Relaxed/Acquire/Release/SeqCst |

## Async

| Rule | Guideline |
|------|-----------|
| Sync for CPU-bound | Async is for I/O |
| Don't hold locks across await | Use scoped guards |

## Macros

| Rule | Guideline |
|------|-----------|
| Avoid unless necessary | Prefer functions/generics |
| Follow Rust syntax | Macro input should look like Rust |

## Deprecated → Better

| Deprecated | Better | Since |
|------------|--------|-------|
| `lazy_static!` | `std::sync::OnceLock` | 1.70 |
| `once_cell::Lazy` | `std::sync::LazyLock` | 1.80 |
| `std::sync::mpsc` | std mpsc is fine (crossbeam-based since 1.67); `crossbeam::channel` for select/MPMC | - |
| `std::convert::Infallible` | `!` (now `type Infallible = !`) | 1.100 |
| `cfg_if!` crate | `cfg_select!` | 1.95 |
| `assert!(matches!(..))` | `std::assert_matches::assert_matches!` | 1.96 |
| `scopeguard` crate | `core::mem::DropGuard` | 1.100 |
| `std::i32::MAX`, `std::f64::EPSILON` | `i32::MAX`, `f64::EPSILON` | deprecated 1.99 |
| `std::char::from_u32` etc. | `char::from_u32` | deprecated 1.97 |
| `#[no_mangle]`, `extern "C" {}` | `#[unsafe(no_mangle)]`, `unsafe extern "C" {}` | edition 2024 |
| `failure`/`error-chain` | `thiserror`/`anyhow` | - |
| `try!()` | `?` operator | 2018 |

## Quick Reference

```
Naming: snake_case (fn/var), CamelCase (type), SCREAMING_CASE (const)
Format: rustfmt (just use it)
Docs: /// for public items, //! for module docs
Lint: [lints.clippy] all/pedantic = "warn" in Cargo.toml; [lints.cargo] unused_dependencies (1.100)
Modern idioms (never type, let chains, if-let guards): see rust-modern-features
```

Claude knows Rust conventions well. These are the non-obvious Rust-specific rules.
