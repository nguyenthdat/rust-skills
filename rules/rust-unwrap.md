---
name: rust-unwrap
description: "Rust (Google style): pattern matching over check-then-unwrap; `expect(\"invariant\")` over `unwrap()`; Option/Result for recoverable failures."
# TTSR: Google Rust Style Guide, "Prefer pattern matching over
# check-then-unwrap", "Prefer expect over unwrap", "Reporting errors".
# Conditions: `.unwrap()` and an empty `.expect("")`.
condition:
  - '\.unwrap\(\)'
  - '\.expect\([ \t]*""[ \t]*\)'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: `unwrap` (Google Rust Style Guide)

- **Prove it instead of asserting it.** When the data already guarantees success, use pattern matching so the compiler checks the guarantee: `if let Some(v) = opt { … }`, `let Some(v) = opt else { … };`, `match`, or `?`. Do not write `if opt.is_some() { opt.unwrap() }`.
- **If a panic really is required** because the type system cannot express the invariant, use `expect("…")` and state the invariant, not the operation: `expect("config validated in load()")`, not `expect("failed to unwrap")`. An empty `expect("")` is no better than `unwrap()`.
- **Recoverable failures are not panics.** I/O, parsing, network, and user input go through `Option`/`Result` and `?`.

Tests and developer-only tools may panic on failure, but use `expect` there when a bare `unwrap()` would make the failure hard to trace. When that holds, move on.
