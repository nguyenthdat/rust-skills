---
name: rust-public-api
description: "Rust (Google style): public API stability: `#[non_exhaustive]` on shared pub enums and all-pub structs, `#[must_use]` where ignoring the result is a bug, `const fn` only when it stays const."
# TTSR: Google Rust Style Guide, "Other Rust features" (non_exhaustive,
# must_use, const fn) and "Don't omit patterns on non_exhaustive enums".
# Conditions: `pub enum` with no `#[non_exhaustive]` between it and the
# previous `;`/`}`; `pub const fn`. All-pub structs are left to the text:
# most are DTOs where the struct-literal exception applies.
condition:
  - '(?m)^(?=[ \t]*pub[ \t]+enum\b)(?<!#\[non_exhaustive\][^;}]*)[ \t]*pub[ \t]+enum[ \t]+\w+'
  - '\bpub[ \t]+const[ \t]+fn\b'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: public API stability (Google Rust Style Guide)

These rules apply to items in the public API of a library used by other teams or crates. Binaries and crate-internal `pub` items can skip them.

- **Mark public enums `#[non_exhaustive]`** so new variants do not break downstream `match` statements (clippy `exhaustive_enums`). Exceptions:
  - The enum will never change: an existing ontology (`enum Color { Red, Green, Blue }`), or a two-state enum that could have been a `bool`.
  - You commit to fixing all downstream breakage yourself.
- **Mark structs whose fields are all public `#[non_exhaustive]`**, because adding any field later breaks struct literals. Exceptions:
  - The fields describe an existing ontology (`Rectangle`).
  - You commit to fixing all downstream breakage yourself.
  - Callers should be able to use `Opts { a: 1, ..Default::default() }` (options structs).
- **Add `#[must_use]`** to functions whose result the caller cannot correctly ignore (pure functions, as `HashMap::get` is) and to functions whose return value reports an error or other critical information, unless the return type is already `#[must_use]` (`Result` is). Mark a type `#[must_use]` when such functions return it: error types, and lazily evaluated computations that do nothing until the value is used.
- **Use `pub const fn` only if it will stay `const`**, or if you can commit to helping clients migrate when it changes. Removing `const` is a breaking change.
- **When matching someone else's `#[non_exhaustive]` enum**, list every known variant and put the ones you ignore in the wildcard arm (`Shape::Triangle | _ => {}`). If you care about only specific variants, use `if let` or `let … else`, unless a `match` is cleaner.

For application or crate-internal types, or ones that fall under a listed exception, move on.
