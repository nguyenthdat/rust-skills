---
name: rust-never-type
description: "Rust 1.100+: write the never type `!` instead of `Infallible`/empty enums; `let Ok(v) = r;` instead of `match e {}` plumbing."
# TTSR: rust-skills `rust-modern-features` (never type stable in 1.100.0,
# 2026-11-12; `Infallible` is now `type Infallible = !;`). Inherits the
# profile-wide `interruptMode: never`: one reminder after the tool result.
# Conditions: `Infallible` anywhere, an empty-match on an Err binding
# (`Err(e) => match e {}`, `|e| match e {}`), and an empty enum declaration
# used as a hand-rolled uninhabited type.
condition:
  - '\bInfallible\b'
  - 'Err\([ \t]*(\w+)[ \t]*\)[ \t]*=>[ \t]*match[ \t]+\1[ \t]*\{[ \t]*\}'
  - '\|[ \t]*(\w+)[ \t]*\|[ \t]*match[ \t]+\1[ \t]*\{[ \t]*\}'
  - '(?m)^[ \t]*(?:pub(?:\([^)\n]*\))?[ \t]+)?enum[ \t]+\w+[ \t]*\{[ \t]*\}'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: the never type `!` (stable since 1.100)

`!` is a first-class type and `std::convert::Infallible` is now just `type Infallible = !;`.

| Instead of | Write |
|---|---|
| `type Err = Infallible;` | `type Err = !;` |
| `Result<T, Infallible>` | `Result<T, !>` |
| `match r { Ok(v) => v, Err(e) => match e {} }` | `let Ok(v) = r;` (irrefutable) |
| `pub enum Void {}` as an "impossible" type | `!` |

- **Fallback changed in every edition:** an unconstrained type from a diverging expression now falls back to `!`, not `()`. If you write `create()?;` where `create<T: Default>()` is generic, the compiler infers `!` and reports `E0277: !: Default`. Annotate it: `create::<()>()?;`.
- `Result<(), !>` is not `#[must_use]`-noisy, and `!` implements `AsRef<T>`/`AsMut<T>`.

Keep `Infallible` when the crate's `rust-version` is below 1.100, when you are matching an existing public API's wording, or when the empty enum carries its own meaning (a marker type with trait impls). In those cases, move on.
