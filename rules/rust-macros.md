---
name: rust-macros
description: "Rust (Google style): prefer functions, types, and traits over macros; `macro_rules!` over code generation; proc macros last."
# TTSR: Google Rust Style Guide, "Macros".
# Conditions: a `macro_rules!` definition or a proc-macro entry point.
condition:
  - '\bmacro_rules![ \t]*\w+'
  - '#\[proc_macro(?:_derive|_attribute)?\b'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: macros (Google Rust Style Guide)

When expressing an API, prefer these in order:

1. Plain functions, types, traits, generics, and modules.
2. `macro_rules!`.
3. Other code generation (build scripts, genrules).
4. Procedural macros.

Before keeping a macro, check that a function, generic, or trait cannot do the job. Macros weaken IDEs, code search, and refactoring tools. They create local dialects that readers must learn, and they make large-scale changes harder. When a macro is justified, `macro_rules!` is less fragile than textual code generation for simple repetition; reserve proc macros for what neither can do.

If a macro is clearly needed (variadic syntax, `format!`-style compile-time checks, a DSL the project already uses, the same trait impl repeated across many types), move on.
