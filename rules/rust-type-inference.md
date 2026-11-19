---
name: rust-type-inference
description: "Rust (Google style): annotate the binding instead of using a turbofish; give explicit types to `.into()`/`.parse()`/`.collect()` results."
# TTSR: Google Rust Style Guide, "Don't overuse type inference".
# Conditions: `let x = …::<T>()` turbofish on collect/parse/sum/product where
# `x` could carry the type; `let x = ….into()/.parse()/.collect()/.sum()/
# .product();` with no type on `x`. Closures/blocks (`{`) end the scan.
condition:
  - '\blet[ \t]+(?:mut[ \t]+)?\w+[ \t]*=[^;{}]*?\.(?:collect|parse|sum|product)::<[^;\n]*?>\(\)\??[ \t]*;'
  - '\blet[ \t]+(?:mut[ \t]+)?\w+[ \t]*=[^;{}]*?\.(?:into|parse|collect|sum|product)\(\)[ \t]*\??[ \t]*;'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: explicit types (Google Rust Style Guide)

Write enough types that a reader who knows the codebase can tell what was inferred.

- **Put the type on the binding, not in a turbofish**: `let ids: Vec<u32> = rows.iter().map(|r| r.id).collect();` rather than `.collect::<Vec<u32>>()`, and `let port: u16 = s.parse()?;` rather than `s.parse::<u16>()?`.
- **Annotate** the results of generic-return methods (`.into()`, `.parse()`, `.collect()`, `.sum()`) and of long iterator chains.
- **When the compiler forces a turbofish** in the middle of an expression with no binding to annotate, let `_` stand for the obvious part: `.collect::<Vec<_>>()`.

If the type is already obvious from a nearby annotation, or the expression has no binding to annotate, move on.
