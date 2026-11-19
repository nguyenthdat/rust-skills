---
name: rust-modern-features
description: "Use when writing or reviewing Rust code to apply features stabilized up to Rust 1.100 (stable, 2026-11-12), or when asked what's new / which version a feature needs. Keywords: never type, !, Infallible, Result<T, !>, never type fallback, let chains, if let guard, cfg_select, assert_matches, trait upcasting, async closure, AsyncFn, use<..> precise capturing, naked function, inferred const _, allocator api, Box::new_in, DropGuard, strict_add, array_windows, extract_if, get_disjoint_mut, LazyLock, c-variadic, MSRV, rust-version, edition 2024, 1.86, 1.87, 1.88, 1.89, 1.90, 1.91, 1.92, 1.93, 1.94, 1.95, 1.96, 1.97, 1.98, 1.99, 1.100, 新特性, 最新语法, never 类型"
source: https://doc.rust-lang.org/stable/releases.html
user-invocable: false
---

# Modern Rust (stable 1.100, edition 2024)

> Baseline: **Rust 1.100.0 (2026-11-12)**, `edition = "2024"`, `rust-version = "1.100"`.
> Verified against RELEASES.md, releases.rs and the 1.100 blog post. For anything newer → `rust-learner`.
> There is **no 2027 edition** (released or stabilized) as of 1.100.

## Rule

Write code for the declared MSRV. With `rust-version = "1.100"`, **use the idioms in the right column** —
don't reach for external crates or old workarounds when std/language now has it.

| Instead of | Write (since) |
|------------|---------------|
| `std::convert::Infallible` / custom empty enum | `!` — `Result<T, !>`, `type Err = !` (**1.100**) |
| `match r { Ok(v) => v, Err(e) => match e {} }` | `let Ok(v) = r;` (irrefutable with `!`) |
| nested `if let` / `match` pyramids | let chains `if let Some(a) = x && let Ok(b) = f(a) && b > 0` (**1.88, edition 2024**) |
| `Some(x) if matches!(..)` + nested match in arm | `if let` guard: `Some(x) if let Ok(y) = parse(x) => ..` (**1.95**) |
| `cfg_if::cfg_if!` | `cfg_select! { unix => {..}, windows => {..}, _ => {..} }` (**1.95**) |
| `assert!(matches!(v, P))` | `use std::assert_matches;` then `assert_matches!(v, P)` / `debug_assert_matches!` (**1.96**, not in prelude) |
| `scopeguard` crate | `core::mem::DropGuard` (with `dismiss`) (**1.100**) |
| `allocator-api2` / nightly `allocator_api` | `Box::new_in`, `Vec::new_in`, `Vec::with_capacity_in` (**1.100**, only Box/Vec) |
| `lazy_static!` / `once_cell` | `LazyLock` / `LazyCell` / `OnceLock` (1.70/1.80); `get_mut`/`force_mut` (**1.94**) |
| `as_any()` hacks to go `&dyn Sub → &dyn Super` | trait upcasting coercion (**1.86**) |
| `split_at_mut` gymnastics for 2 indices | `slice.get_disjoint_mut([i, j])`, `HashMap::get_disjoint_mut` (**1.86**) |
| `retain` + collect-removed loops | `Vec/HashMap/HashSet/BTreeMap::extract_if` (1.87/1.88/1.91) |
| `if v.last()… { v.pop() }` | `Vec::pop_if` (**1.86**), `VecDeque::pop_front_if/pop_back_if` (**1.93**) |
| `windows(N)` + `try_into().unwrap()` | `array_windows::<N>()` (**1.94**), `as_chunks::<N>()` (**1.88**) |
| `checked_add(..).expect("overflow")` | `strict_add` & `strict_*` family (**1.91**) |
| `x % n == 0` (unsigned) | `x.is_multiple_of(n)` (**1.87**) |
| `c.set(c.get() + 1)` | `Cell::update(|v| v + 1)` (**1.88**), `Atomic*::update` (**1.95**) |
| `fs2`/`fd-lock` for file locks | `File::lock / try_lock / lock_shared / unlock` (**1.89**) |
| `os_pipe` crate | `std::io::pipe()` (**1.87**) |
| `opt.map_or(Default::default(), f)` | `map_or_default(f)` on `Option`/`Result` (**1.98**) |
| `s.strip_prefix(p).unwrap_or(s)` | `s.trim_prefix(p)` / `trim_suffix` (**1.100**) |
| moving out of a `Box` then re-allocating | `let (val, slot) = Box::take(boxed);` keeps the allocation as `Box<MaybeUninit<T>>` (**1.100**); `Box::map`/`Rc::map`/`Arc::map` |
| `b = !b` | `b.toggle()` (**1.100**) |
| hand-rolled `struct Wrapper` just for `Display` | `std::fmt::from_fn(|f| ..)` (**1.93**) |
| `itoa` crate for no-alloc int formatting | `core::fmt::NumBuffer` + `{int}::format_into` (**1.98**) |
| `#[cold]` helper fn to mark a branch | `core::hint::cold_path()` (**1.95**) |
| `std::i32::MAX`, `std::char::from_u32` | `i32::MAX`, `char::from_u32` (legacy modules deprecated 1.97/1.99) |
| `[0u8; 3]` with repeated length | `let a: [u8; _] = [0; 3];` inferred const args (**1.89**) |
| `Box<dyn Fn() -> Pin<Box<dyn Future>>>` | async closures `async \|x\| ..` + `impl AsyncFn(X) -> Y` (1.85) |
| implicit captures in RPIT | `impl Trait + use<'a, T>` precise capturing, also in traits (**1.87**) |
| `#[no_mangle]`, `extern "C" {}` | `#[unsafe(no_mangle)]`, `unsafe extern "C" {}` (edition 2024) |

## The Never Type `!` (stable in 1.100)

```rust
use std::convert::Infallible;
use std::str::FromStr;

struct Name(String);

impl FromStr for Name {
    type Err = !;                       // was: Infallible
    fn from_str(s: &str) -> Result<Self, !> { Ok(Name(s.to_owned())) }
}

fn into_ok<T>(r: Result<T, !>) -> T {
    let Ok(v) = r;                      // irrefutable: Err(!) is uninhabited
    v
}

fn same(x: Infallible) -> ! { x }       // Infallible is now `type Infallible = !;`
```

Key facts:
- `!` is a first-class type: usable in generics, fields, `Result<T, !>`, `Vec<!>`, associated types.
- `std::convert::Infallible` is now **a type alias for `!`** — existing `Infallible` APIs accept `!` and vice-versa.
- `unused_must_use` doesn't fire on `Result<(), !>` (1.92); `Result<T, !>` is treated like `T` for `must_use` (1.97).
- `!` implements `AsRef<T>`/`AsMut<T>` (1.100) besides the usual traits.
- In new code prefer writing `!`; `Infallible` remains valid for older MSRVs.

### Breaking change: never-type fallback is `!` in **all editions** (1.100)

An unconstrained inference variable produced by a diverging expression used to fall back to `()`; now it's `!`.

```rust
fn create<T: Default>() -> Result<T, String> { Ok(T::default()) }

fn run() -> Result<(), String> {
    create()?;          // ❌ 1.100: error[E0277]: the trait bound `!: Default` is not satisfied
    create::<()>()?;    // ✅ annotate the type explicitly
    Ok(())
}
```
Fix pattern: **annotate the type** (`::<()>`, `let _: () = ..`). This was warned since 1.89 (incl. dependencies)
and deny-by-default since 1.92 (`dependency_on_unit_never_type_fallback`, `never_type_fallback_flowing_into_unsafe`).
Also watch: new `unreachable_code` warnings after expressions that now have type `!`.

## Language Features by Version (1.86 → 1.100)

| Version (date) | Language / compiler | Notable std / tooling |
|---|---|---|
| **1.86** (2025-04-03) | trait upcasting; `#[target_feature]` on safe fns | `get_disjoint_mut`, `Vec::pop_if`, `OnceLock::wait` |
| **1.87** (2025-05-15) | `use<..>` in trait RPITIT; `asm_goto` | `io::pipe`, `Vec::extract_if`, `is_multiple_of`, `cast_signed/unsigned` |
| **1.88** (2025-06-26) | **let chains** (2024); **naked fns** `#[unsafe(naked)]`; `cfg(true/false)` | `Cell::update`, `as_chunks`, `HashMap::extract_if`; Cargo cache auto-GC |
| **1.89** (2025-08-07) | inferred const args `_`; `repr(u128/i128)`; AVX-512; lint `mismatched_lifetime_syntaxes` | `File::lock`, `Result::flatten`, `NonNull::from_ref` |
| **1.90** (2025-09-18) | `lld` default on x86_64 Linux | `checked_sub_signed`; `cargo publish --workspace` |
| **1.91** (2025-10-30) | aarch64 Windows Tier 1; lints `dangling_pointers_from_locals` | `strict_*` arithmetic, `Path::file_prefix`, `Duration::from_mins/hours`, `BTreeMap::extract_if`; `build.build-dir` |
| **1.92** (2025-12-11) | never-type fallback lints → **deny** | `RwLockWriteGuard::downgrade`, `Box::new_zeroed` |
| **1.93** (2026-01-22) | `#[cfg]` on `asm!` lines; `deref_nullptr` → deny | `fmt::from_fn`, `Vec::into_raw_parts`, `VecDeque::pop_*_if` |
| **1.94** (2026-03-05) | `unused_visibilities` lint; std macros via prelude | `array_windows`, `LazyLock::get_mut`, `Peekable::next_if_map`; Cargo config `include`, TOML 1.1 |
| **1.95** (2026-04-16) | **`if let` guards**; **`cfg_select!`** | `hint::cold_path`, `Atomic*::update`, `Vec::push_mut`, `core::range` |
| **1.96** (2026-05-28) | never coerced in tuples | **`std::assert_matches::assert_matches!`**, new `core::range::Range*` (Copy) types |
| **1.97** (2026-07-09) | v0 symbol mangling default; `pin!` no deref-coercion | int `bit_width`/`highest_one`; `std::char` consts deprecated; Cargo **`build.warnings`** |
| **1.98** (2026-08-20) | `repr(transparent)` stricter; lint `c_void_returns` | `map_or_default`, `NumBuffer`/`format_into`, algebraic floats, `str::substr_range` |
| **1.99** (2026-10-01) | **C-variadic fn definitions**; attr macros on `mod foo;` | `Vec::into_parts/from_parts`, `Box::into_non_null`, `fs::set_times`; legacy int modules deprecated; Cargo `debug` profile |
| **1.100** (2026-11-12) | **never type `!`**; fallback `!` in all editions; **allocator API** (Box/Vec); `extern "custom"`; `box` patterns removed | `mem::DropGuard`, `bool::toggle`, `Box::take`, `Box/Rc/Arc::map`, `trim_prefix/suffix`, `Vec::from_fn`; **`[lints.cargo]`** (`unused_dependencies`), `min-publish-age` cooldowns |

## Still Unstable at 1.100 (do NOT use in production — Google style G.FEA.2)

`gen` blocks, `try` blocks, specialization / `min_specialization`, default field values, ergonomic clones / `use`
closures, return-type notation, `impl Trait` in associated types / TAIT, async drop, `async fn` in `dyn` traits,
`super let`, deref patterns, pin ergonomics, arbitrary self types, unsafe fields, `f16`/`f128`, `const` trait impls,
cargo-script frontmatter. Use the stable workarounds (`async-trait`/`trait-variant` for dyn async, boxing, etc.).

## Cargo.toml Baseline (1.100)

```toml
[package]
edition = "2024"
rust-version = "1.100"

[lints.rust]
unsafe_code = "warn"
missing_docs = "warn"            # libraries

[lints.clippy]
all = { level = "warn", priority = -1 }
pedantic = { level = "warn", priority = -1 }

[lints.cargo]                    # Cargo lint system (1.100)
unused_dependencies = "warn"
```

```toml
# .cargo/config.toml
[build]
warnings = "deny"                # CI: deny warnings without RUSTFLAGS=-Dwarnings (1.97)

[registry]
global-min-publish-age = "3 days"  # supply-chain cooldown for new releases (1.100)
```

Notes:
- `cargo install` now uses the packaged lockfile by default (1.100); `--locked` is a no-op.
- Lowering `rust-version` below 1.100 means: no `!` in type position (use `Infallible`), no `DropGuard`,
  no allocator API; below 1.95 no `if let` guards / `cfg_select!`; below 1.88 no let chains.

## Related Skills

| When | See |
|------|-----|
| Version questions beyond 1.100 / live data | rust-learner (rust-changelog agent) |
| Style & lint policy | coding-guidelines (`google-style.md`) |
| Error types (`!` vs `Infallible`) | m06-error-handling |
| Allocator API / custom allocation | m10-performance, m02-resource |
| C-variadics, naked fns, `extern` | unsafe-checker |
