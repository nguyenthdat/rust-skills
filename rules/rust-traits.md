---
name: rust-traits
description: "Rust (Google style): derive when possible, implement the std trait instead of a look-alike method, `Default` beside `new()`, `is_empty` beside `len`, operators only for math."
# TTSR: Google Rust Style Guide, "Traits", plus "Implement expected API sets
# together" and "Avoid overloading for non-mathematical purposes".
# Conditions: a hand-written impl of a derivable trait; inherent `to_string`
# or `from_str`; a std-trait method signature (`default() -> Self`,
# `next(&mut self) -> Option`, `deref(&self)`, …) inside an inherent `impl X {`;
# `pub fn len(&self)` with no `fn is_empty` in the snapshot; `pub fn new() ->
# Self` with no `Default` in the snapshot; an operator-trait impl.
condition:
  - '\bimpl(?:<[^>]*>)?[ \t]+(?:(?:std|core)::(?:clone|marker|default|cmp|hash|fmt)::)?(?:Clone|Copy|Default|PartialEq|Eq|PartialOrd|Ord|Hash|Debug)[ \t]+for\b'
  - '\bfn[ \t]+to_string[ \t]*\([ \t]*&self[ \t]*\)[ \t]*->[ \t]*String\b'
  - '\bfn[ \t]+from_str[ \t]*\([ \t]*\w+[ \t]*:[ \t]*&(?:''\w+[ \t]+)?str[ \t]*\)[ \t]*->[ \t]*(?:Self|Option[ \t]*<[ \t]*Self[ \t]*>)'
  - '\bfn(?<!\basync[ \t]+fn)[ \t]+(?:default[ \t]*\([ \t]*\)[ \t]*->[ \t]*Self\b|next[ \t]*\([ \t]*&mut[ \t]+self[ \t]*\)[ \t]*->[ \t]*Option\b|(?:eq|ne|cmp|partial_cmp)[ \t]*\([ \t]*&self[ \t]*,[^)]*&Self[ \t]*\)|clone[ \t]*\([ \t]*&self[ \t]*\)[ \t]*->[ \t]*Self\b|into_iter[ \t]*\([ \t]*self[ \t]*\)|deref[ \t]*\([ \t]*&self[ \t]*\)|as_ref[ \t]*\([ \t]*&self[ \t]*\)[ \t]*->[ \t]*&|as_mut[ \t]*\([ \t]*&mut[ \t]+self[ \t]*\)[ \t]*->[ \t]*&mut\b|hash[ \t]*<|from_iter[ \t]*<)(?<=\bimpl(?:<[^>{]*>)?[ \t]+(?:(?!\bfor\b)[\w:<>, ''])+\{(?:(?!\bimpl\b)[\s\S])*)'
  - '^(?![\s\S]*\bfn[ \t]+is_empty\b)[\s\S]*?\bpub[ \t]+fn[ \t]+len[ \t]*\([ \t]*&self[ \t]*\)'
  - '^(?![\s\S]*(?:\bimpl(?:<[^>]*>)?[ \t]+(?:(?:std|core)::default::)?Default[ \t]+for\b|#\[derive\([^)]*\bDefault\b))[\s\S]*?\bpub[ \t]+(?:const[ \t]+)?fn[ \t]+new[ \t]*\([ \t]*\)[ \t]*->[ \t]*Self\b'
  - '\bimpl(?:<[^>]*>)?[ \t]+(?:(?:std|core)::)?(?:ops::)?(?:Add|Sub|Mul|Div|Rem|Neg|Not|BitAnd|BitOr|BitXor|Shl|Shr)(?:Assign)?(?:[ \t]*<[^>]*>)?[ \t]+for\b'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: traits (Google Rust Style Guide)

- **Derive instead of hand-writing** `Clone`, `Copy`, `Debug`, `Default`, `PartialEq`/`Eq`, `PartialOrd`/`Ord`, or `Hash` whenever `#[derive(...)]` behaves the same. Hand-write an impl only for different semantics (a redacting `Debug`, non-zero defaults, comparing a subset of fields) or to avoid the bounds that derive adds to type parameters.
- **Implement the std trait instead of a look-alike method**: `impl FromStr` rather than an inherent `from_str`, `impl Display` rather than an inherent `to_string`, and likewise `Default`, `Iterator`, `PartialEq`, `Ord`, `Deref`, `AsRef`, `IntoIterator` (clippy `should_implement_trait`). Use `ops::*` traits only when the method really is arithmetic or logic. An inherent method kept beside the trait impl for discoverability is fine.
- **Implement `Default` for any type with `pub fn new() -> Self`** (clippy `new_without_default`). Bag-of-options structs with public fields also implement `Default`, so callers can write `Opts { x: 1, ..Default::default() }`.
- **Pair `pub fn len()` with `is_empty()`** (clippy `len_without_is_empty`).
- **Overload operators only for their mathematical or logical meaning**: `Add` for vectors or money, never `+` as shorthand for "push" or "merge".
- **Pair an important `From` impl with an explicit constructor** such as `new()` or `from_foo()`.
- **Which traits to implement**:
  - `Clone`: most types, except handles to external resources.
  - `Copy`: when the type will clearly stay `Copy`; prefer it for error types.
  - `Default`: when there is a natural default state.
  - `PartialEq`/`PartialOrd`: to enable `==` and `<`. Add `Eq`/`Ord` only when reflexivity and total ordering really hold.
  - `Hash` + `Eq` (+ `Ord`): types likely to become map keys.
  - Types with public fields: consider implementing the traits that all their fields implement.
  - Libraries anticipate these needs; application code adds impls when it needs them.
- **Define a trait only** for generic bounds, for `impl Trait`/`dyn Trait`, or as an extension trait. Never define one just to organize methods; use an inherent `impl` instead.

If the impl differs from what derive would generate, the method sits inside an `impl Trait for …` block, or the `is_empty`/`Default` already exists outside the lines you edited, move on.
