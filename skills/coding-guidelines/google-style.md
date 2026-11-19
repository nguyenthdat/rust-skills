# Google Rust Style Guide — Condensed Rules

> Source: https://google.github.io/styleguide/rustguide.html (retrieved 2026-11-19; page has no last-updated date)
>
> Scope: general rules below apply everywhere. Rules the guide tags `[chrome]`, `[Fuchsia]`, `[Android]`
> are codebase-specific and are listed separately at the end — do NOT apply them to normal projects.
> The guide defers to the official Rust Style Guide (rustfmt) and the Rust API Guidelines unless it says otherwise.
>
> Strength: the guide uses "should / prefer / avoid / don't" rather than RFC keywords.
> Library code: treat API-design rules as binding. Application code: strong recommendation, favour simplicity.

---

## G.ORG — Code Organization

| ID | Rule |
|----|------|
| G.ORG.1 | Name source files / leaf modules after their primary item: `Foo` lives in `foo.rs`. |
| G.ORG.2 | Non-leaf modules contain only `mod` declarations and re-exports (plus a little "glue"). `mod tests` and `#[cxx::bridge]` modules count as leaf modules. |
| G.ORG.3 | **No glob imports.** Allowed: `use super::*` in inline modules (e.g. `mod tests`), `use MyEnum::*` inside functions about that enum, `pub use submodule::*` in non-leaf modules. Don't define your own preludes. |
| G.ORG.4 | One `use` per source module, no nested paths in braces (rustfmt `imports_granularity = "Module"`). |
| G.ORG.5 | Import traits locally inside the function that needs them, unless used throughout the file. |
| G.ORG.6 | Call free functions qualified by their module: `mem::swap(a, b)`, not an imported `swap(a, b)`. |
| G.ORG.7 | Import types and use them unqualified (`Regex`, not `regex::Regex`). Exception: ambiguous names → `io::Error` or `use std::io::Error as IoError`. |

```rust
// Good
use foo::Foo;
use foo::bar::{Bar, Baz};
use std::mem;

// Bad
use foo::{Foo, bar::Bar};   // nested path inside braces
use std::mem::transmute;    // then calling transmute(x) unqualified

fn parse(s: &str) -> Result<u8, std::num::ParseIntError> {
    use std::str::FromStr; // locally scoped trait import
    u8::from_str(s)
}
```

## G.FMT — Formatting

| ID | Rule |
|----|------|
| G.FMT.1 | Always run rustfmt. |
| G.FMT.2 | `#[rustfmt::skip]` only for deliberate manual alignment (matrices, tables) or generated code — never for personal taste. |

Recommended `rustfmt.toml` (options marked * need nightly rustfmt; skip them if you must build on stable):

```toml
max_width = 100
imports_granularity = "Module"   # *
hex_literal_case = "Lower"       # *
format_macro_matchers = true     # *
```

## G.OWN — Ownership & Borrowing

| ID | Rule |
|----|------|
| G.OWN.1 | **Avoid `Rc`/`Arc`.** Prefer unique ownership + borrows; use shared ownership only when the alternative is unreasonably complex (justify in review). |
| G.OWN.2 | Prefer borrows over deep `.clone()`. Clone only for a real deep copy, or pragmatically when borrowing becomes excessively complex. (Cloning `Rc`/`Arc` is fine.) |

## G.API — Function Inputs & Outputs

| ID | Rule |
|----|------|
| G.API.1 | Accept the data, not the wrapper: `&T` not `&Box<T>`, `&[T]` not `&Vec<T>`, `&str` not `&String`, `Option<&T>` not `&Option<T>` (clippy `borrowed_box`, `ptr_arg`). |
| G.API.2 | Prefer returning an iterator (`impl Iterator<Item = T>`) over `Vec<T>` — unless callers need a Vec, it's naturally built as one, will be indexed, or it's application code where Vec is simpler. |
| G.API.3 | Reusable library code that only iterates should accept `impl IntoIterator<Item = T>`; application code may take `&[T]`. |
| G.API.4 | `impl Into<T>` params only in library APIs where call-site ergonomics are paramount. Application code should almost never use them. |
| G.API.5 | Prefer an `enum` over a `bool` parameter when the meaning isn't obvious; if you keep `bool`, annotate the call site: `connect(/* retry= */ true)`. |
| G.API.6 | Errors: `Option` for obvious/no-detail failures; `Result` for recoverable failures that need context (distinguishable modes); `panic!` only for bugs / violated invariants. |

## G.TYP — Types

| ID | Rule |
|----|------|
| G.TYP.1 | Newtype (not type alias) when semantics differ, it needs its own API, the representation may change, or values are easy to mix up (`Day`, `Month`, `Year`). |
| G.TYP.2 | Prefer named fields over tuples / multi-field tuple structs. Single-field newtypes are fine. |

## G.TRT — Traits

| ID | Rule |
|----|------|
| G.TRT.1 | Implement std traits instead of look-alike inherent methods (`impl FromStr`, not `fn from_str`) — clippy `should_implement_trait`. |
| G.TRT.2 | Use `#[derive(...)]` whenever it expresses the impl. |
| G.TRT.3 | Libraries: prefer generics / `impl Trait` over `dyn Trait` in parameters, unless code size matters. |
| G.TRT.4 | Define a trait only for generic bounds, `impl Trait` / `dyn Trait`, or extension traits. **Don't use traits merely to organize code** — use inherent `impl`. |
| G.TRT.5 | Extension traits are named `FooExt`; the team owning `Foo` should usually own `FooExt`. |
| G.TRT.6 | `Copy` when it will clearly stay `Copy` (error types: whenever possible). `Clone` for most types except external-resource handles. `Default` when there's a `new()` or default state (`new_without_default`). Likely map keys: `Hash + Eq` (+ `Ord` if total order). |
| G.TRT.7 | If a `From` impl is a primary constructor, also provide an inherent `new()` / `from_foo()`. |
| G.TRT.8 | All-public-field "options" structs implement `Default` so callers can write `Opts { a: 1, ..Default::default() }`. |

## G.CON — Concurrency

| ID | Rule |
|----|------|
| G.CON.1 | Never reach into a poisoned mutex (`PoisonError::into_inner`) to get at the data. |

## G.MAC — Macros

Preference order: **plain functions/types/traits → `macro_rules!` → other codegen → proc macros.** Macros hurt IDEs, code search and large-scale refactors.

## G.NAM — Naming

| ID | Rule |
|----|------|
| G.NAM.1 | `as_`/`to_`/`into_` prefixes with matching `self` conventions (clippy `wrong_self_convention`). |
| G.NAM.2 | Descriptive lifetime names (`'src`, `'conn`); `'_` where possible; single letters only for local input→output relations or `for<'a>`, max two. |
| G.NAM.3 | Short generic names hinting at the main bound (`I: Iterator`, `F: Fn`); `T`/`U` for largely unbounded APIs. |
| G.NAM.4 | Constructors: `new()`, `new_with_foo()` for variants, `try_new()` if it returns `Result`; conversions `from_foo()` + `From` impl. |
| G.NAM.5 | Avoid non-obvious abbreviations; don't drop letters inside words. Widely-known ones (`IpAddr`) are fine. |
| G.NAM.6 | Accessors: `foo()` (→ `&T`, or `T` if `Copy`), `foo_mut()`, `set_foo(T)`, `take_foo()` (leave default/None, return old), `replace_foo(T) -> T`, `with_foo(|f| ..)` / `with_mut_foo(|f| ..)` for values behind `Mutex`/`RefCell` (don't leak guard types). |

## G.DOC — Documentation

| ID | Rule |
|----|------|
| G.DOC.1 | Doc-comment public APIs (`missing_docs` lint helps). Crate should have examples covering its functionality. |
| G.DOC.2 | Always document `# Panics` for inputs that can panic, and `# Safety` invariants for unsafe fns/traits. |
| G.DOC.3 | Full sentences, Markdown. `///` for items, `//!` at file top for modules. `//` for implementation notes/TODOs (mixing `///` and `//` is encouraged). |
| G.DOC.4 | Don't restate the name or explain basic language concepts — omit the comment if it adds nothing. |
| G.DOC.5 | Write docs directly in source, not generated from macros. |

## G.FEA — Other Language Features

| ID | Rule |
|----|------|
| G.FEA.1 | `unsafe` only if it can't be done safely; wrap it in a safe abstraction isolated from business logic (ideally its own small crate); every block gets `// SAFETY:`; justify why not safe (benchmarks if "performance"). |
| G.FEA.2 | **No unstable (nightly) features** in production code. |
| G.FEA.3 | `len()` ⇒ also `is_empty()` (clippy `len_without_is_empty`). |
| G.FEA.4 | Treat `#[doc(hidden)]` items you don't own as private. |
| G.FEA.5 | Mark public fns `const` only if you commit to keeping them `const` (removing it is breaking). |
| G.FEA.6 | Operator overloading only for mathematical/logical meaning, not as syntax sugar. |
| G.FEA.7 | `#[non_exhaustive]` on public enums and on public structs with public fields consumed by other teams/crates — unless the set is truly fixed (`Color`, two-state enums, `Rectangle`) or you want functional record update (clippy `exhaustive_enums`). |
| G.FEA.8 | `#[must_use]` on fns whose result must be used (pure getters like `HashMap::get`, critical status) — not when the return type is already `#[must_use]` (`Result`). Put it on error types and lazy computations. |
| G.FEA.9 | No code before `main()` (no `ctor` crate). |

## G.IMP — Implementation

| ID | Rule |
|----|------|
| G.IMP.1 | Don't over-rely on inference: annotate `let` bindings (prefer over turbofish), especially after `.into()`, `.parse()`, `.collect()` and long iterator chains. `_` inside a forced turbofish is fine. |
| G.IMP.2 | Pattern-match instead of check-then-unwrap (`if let Some(v) = opt`, not `if opt.is_some() { opt.unwrap() }`). |
| G.IMP.3 | `expect("reason")` over `unwrap()` when an invariant can't be expressed in types. |
| G.IMP.4 | On `#[non_exhaustive]` enums, list all known variants; group unneeded ones with the wildcard arm. Use `if let` / `let ... else` when you care about one variant. |
| G.IMP.5 | Give assertions context: `assert!(refund > 0, "Refund must be positive, got {refund:?}")`. Explain misuse vs. bug; don't just restate the condition. |
| G.IMP.6 | Use a `for` loop when clearer than combinators: avoid long chains, don't build chains in the `for` header, avoid `for_each` and side-effecting closures. Short combinators (`iter().any(..)`) are fine. |
| G.IMP.7 | Don't call `panic::catch_unwind` (only as never-relied-upon defense in depth; useless under `panic = "abort"`). |
| G.IMP.8 | Use `format!` / `write!` to concatenate >2 strings or substitute into templates, not `a + b + c` or `str::replace`. |

---

## Codebase-Specific Rules (do NOT apply by default)

| Tag | Rule |
|-----|------|
| [chrome] | Small crates, flat file layout, `chromium::import!()`, no `async`/`.await`, no std threads, `rust_gtest_interop` tests, `git cl format`. |
| [Fuchsia] | `fuchsia_sync::Mutex`, `#[fuchsia::test]`. |
| Chrome/Android/Fuchsia | `panic = "abort"` everywhere. |

## Relevant Clippy Lints (from the guide)

`borrowed_box`, `ptr_arg`, `should_implement_trait`, `wrong_self_convention`, `new_without_default`,
`len_without_is_empty`, `exhaustive_enums` (restriction — enable per-crate), plus rustc `missing_docs`.

```toml
[lints.rust]
missing_docs = "warn"           # libraries

[lints.clippy]
exhaustive_enums = "warn"       # public libraries consumed across teams
exhaustive_structs = "warn"
```
