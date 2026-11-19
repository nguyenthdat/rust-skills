---
name: rust-imports
description: "Rust (Google style): no glob imports, one module per `use`, types imported and used unqualified, functions called through their module."
# TTSR: Google Rust Style Guide (google.github.io/styleguide/rustguide.html),
# "Code organization". Like every rules/rust-*.md rule it watches Rust source
# written by edit/write and inherits the profile-wide `interruptMode: never`:
# a match leaves one reminder after the tool result, once per session.
# Conditions: glob `use` (minus `use super::*`, enum-variant globs such as
# `use Shape::*`, and `pub use` re-exports), `::` inside a `use {…}` group,
# and a CamelCase type reached through 2+ lowercase path segments outside
# `use` statements, `//` lines and string literals
# (`std::collections::HashMap`; single-module `io::Error`, ALL_CAPS consts
# and macro-style `::std::`/`$crate::` stay quiet).
condition:
  - '(?m)^[ \t]*use[ \t]+(?!super::\*[ \t]*;|(?:\w+::)*[A-Z]\w*::\*)[^;]*::\*'
  - '(?m)^[ \t]*(?:pub(?:\([^)\n]*\))?[ \t]+)?use[ \t]+[^;{]*\{[^;}]*::'
  - '(?m)^(?![ \t]*//)[^\n]*?(?<![\w:$])[a-z_][a-z0-9_]*::(?:[a-z_][a-z0-9_]*::)+[A-Z]\w*?[a-z]\w*(?<!^[ \t]*(?:pub(?:\([^)\n]*\))?[ \t]+)?use\b[^;]*)(?<=^(?:[^"\n]*"(?:[^"\\\n]|\\.)*")*[^"\n]*)'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: imports (Google Rust Style Guide)

- **No glob imports** (`use foo::*`). Allowed at your discretion: `use super::*` in an inline module such as `mod tests`; `use MyEnum::*` inside a function or method body that mostly handles that enum; `pub use submodule::*` re-exports in a non-leaf module; merging the same C++ namespace from several crates into one private module (`mod absl { pub use status::absl::*; … }`). Third-party preludes are up to the reviewer. Avoid defining preludes.
- **One module per `use`** (rustfmt `imports_granularity = "Module"`): no `::` inside `{…}`. Write `use foo::Foo;` and `use foo::bar::{Bar, Baz};`, not `use foo::{Foo, bar::Bar};`.
- **Import types and traits, then use them unqualified**: `use regex::Regex;` and then `Regex`, not `regex::Regex` or `std::collections::HashMap` in code. Allowed: one module qualifier or a rename for ambiguous names (`io::Error`, `fmt::Result`, `use std::io::Error as IoError`); large families of types (`hir::…`); macros and generated code; files where importing everything would bloat the import block for little gain.
- **Call functions through their module**: `use std::mem;` and then `mem::take(x)`, not `use std::mem::take;` and then `take(x)`. Allowed: names that are unambiguous on their own, such as C++ interop free functions, or a module name that is prone to collisions.
- **Import traits in the narrowest scope**: put `use std::str::FromStr;` inside the function that needs it, unless the trait's methods are used throughout the file.
- **Module layout**: a leaf module with one primary item is named after it (`Foo` lives in `foo.rs`). A non-leaf module holds only `mod` declarations and re-exports, plus at most a little glue code. A leaf module may still contain `mod test` and binding modules such as `#[cxx::bridge]`.

If the repo's `rustfmt.toml` (`imports_granularity`), its lint config, or the surrounding code already uses a different convention, follow the repo and move on.
