---
name: rust-restricted-features
description: "Rust (Google style): no unstable features, no `catch_unwind`, no poisoned-mutex recovery, no code before `main()`, no use of other crates' `#[doc(hidden)]` items."
# TTSR: Google Rust Style Guide, "No unstable features", "Don't call
# panic::catch_unwind", "Handling poison", "Do not run code before main()",
# "Do not access items marked #[doc(hidden)] that you don't own".
condition:
  - '#!\[feature[ \t]*\('
  - '\bcatch_unwind\b'
  - '\bPoisonError::into_inner\b'
  - '\.(?:lock|read|write|try_lock|try_read|try_write)\(\)[^;]*?\|[ \t]*(\w+)[ \t]*\|[ \t]*\1\.(?:into_inner|get_ref|get_mut)\(\)'
  - '\.(?:lock|read|write|try_lock|try_read|try_write)\(\)[^;]*?\bErr\([ \t]*(\w+)[ \t]*\)[ \t]*=>[ \t]*\{?[ \t\r\n]*\1\.(?:into_inner|get_ref|get_mut)\(\)'
  - '\.clear_poison\(\)'
  - '#\[(?:ctor|dtor)\b|\bctor::|#\[link_section[ \t]*=[ \t]*"[^"]*(?:init_array|mod_init_func|CRT\$X)'
  - '(?<![$\w])(?!(?:self|crate|super)::)[a-z_]\w*::__(?:private|priv|internal|export|reexport|rt|impl|macro_support|hidden)\w*'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: restricted features (Google Rust Style Guide)

- **No unstable features** (`#![feature(...)]`). Code builds on stable Rust unless the project has explicitly approved that specific feature, as some embedded or OS targets do.
- **No `panic::catch_unwind`.** A panic reports a bug: fix the bug instead of recovering from it. The one exception is a defense-in-depth wrapper at a critical entry point of a `panic = "unwind"` build that already works hard to avoid panics and never relies on the catch for correctness.
- **Never reach inside a poisoned mutex**, however tempting the use case (`PoisonError::into_inner`, `.unwrap_or_else(|e| e.into_inner())`, `clear_poison()`). Poisoning means another thread panicked mid-update, and the data may be inconsistent. Let the panic propagate: `.lock().expect("…")`.
- **No code before `main()`**: no `#[ctor]`/`#[dtor]` and no `.init_array`/`__mod_init_func` link sections. Initialize explicitly from `main`, or lazily (`std::sync::LazyLock`).
- **Treat other crates' `#[doc(hidden)]` items as private.** Do not use their `__private` or other `__`-prefixed paths unless you own that crate.

If the project explicitly allows the construct (for example, a nightly crate with approved features), move on.
