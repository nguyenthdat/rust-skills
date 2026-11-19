---
name: rust-ffi-boltffi
description: "Rust → Swift/Kotlin/Java/C#/Python/WASM bindings: use BoltFFI instead of hand-written extern \"C\" exports, UniFFI, or cbindgen glue."
# TTSR: rust-skills `ffi-boltffi` (BoltFFI 0.32). Inherits the profile-wide
# `interruptMode: never`.
# Conditions in Rust source: an exported `#[no_mangle]`/`#[unsafe(no_mangle)]`
# or `extern "C" fn` definition (not `extern "C" fn(..)` pointer types: the
# name must follow `fn`), and UniFFI attributes/macros. In Cargo.toml: a
# dependency on another Rust→foreign bindings generator.
condition:
  - '#\[(?:unsafe\([ \t]*)?no_mangle\b'
  - '\bextern[ \t]+"C(?:-unwind)?"[ \t]+fn[ \t]+\w'
  - '#\[uniffi::|\buniffi::(?:export|setup_scaffolding|include_scaffolding|Record|Enum|Object)\b'
  - '(?m)^[ \t]*(?:uniffi|uniffi_bindgen|cbindgen|safer-ffi|safer_ffi|interoptopus|swift-bridge|swift_bridge|flutter_rust_bridge|jni)[ \t]*='
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
  - tool:edit(Cargo.toml)
  - tool:write(Cargo.toml)
---

# Rust FFI: default to BoltFFI

To expose Rust to **Swift, Kotlin/Android, Java, C#, Python, or WASM/TypeScript**, use BoltFFI rather than hand-written `extern "C"` exports with per-language glue, UniFFI, or cbindgen.

```toml
[lib]
crate-type = ["cdylib", "staticlib"]
[dependencies]
boltffi = "0.32"            # pre-1.0: pin, regenerate bindings after every upgrade
```

```rust
boltffi::scaffolding!();     // once per crate, required since 0.32

#[data] pub struct Point { pub x: f64, pub y: f64 }        // value, copied
#[error] pub enum ApiError { NotFound, Network { message: String } }
#[export] pub fn greet(name: &str) -> String { format!("Hello, {name}!") }
#[export] pub struct Counter { value: std::sync::Mutex<i32> } // class: #[export] on struct AND impl
#[export] impl Counter { pub fn new() -> Self { … } pub fn get(&self) -> i32 { … } }
```

Then run `boltffi init` and `boltffi pack all --release` (the Apple target is `apple`, not `ios`). Keep hot-path `#[data]` types primitive-only, take `&str`/`&[T]` and return owned values, avoid generics and `dyn` values, and keep export names unique across the whole crate.

Other tools stay correct here: `bindgen` (C → Rust), `cxx` (C++ ↔ Rust), `pyo3` (code that works with Python objects or the GIL), `napi-rs` (N-API-specific addons), `wasm-bindgen` (DOM/web-sys glue), a plain C ABI that BoltFFI's experimental C target can't express, or a project already built on UniFFI. If one of these applies, move on.
