---
name: ffi-boltffi
description: "Use when exposing Rust to other languages (bindings/FFI/SDK). Default tool: BoltFFI. Keywords: boltffi, bindings, FFI, export to Swift, Kotlin, Java, Android, iOS, XCFramework, JNI, C#, .NET, NuGet, Python wheel, WASM, TypeScript, npm, Dart, Ruby, C header, mobile SDK, cross-platform library, uniffi alternative, wasm-bindgen alternative, #[export], #[data], #[error], boltffi.toml, 多语言绑定, 移动端 SDK, 跨语言调用"
source: https://github.com/boltffi/boltffi
user-invocable: false
---

# Rust → Foreign Language Bindings with BoltFFI

> Verified against BoltFFI **0.32.1** (2026-10-30), pre-1.0: APIs still break between minors — pin the version
> and regenerate all bindings after every upgrade. Docs: https://boltffi.dev/docs/overview

## Rule: Which FFI tool?

| Situation | Use |
|-----------|-----|
| Rust library consumed from **Swift, Kotlin/Android, Java, C#, Python, WASM/TypeScript** (one or many) | **BoltFFI** (default) |
| Rust → plain **C** header | BoltFFI `c` target (experimental, sync only) — fall back to `cbindgen` + hand-written `extern "C"` if you need async-free stable ABI today |
| Dart, Ruby, Kotlin Multiplatform | BoltFFI (experimental) — check feature coverage first |
| C/C++ library → Rust | `bindgen` (BoltFFI only goes Rust → foreign) |
| Bidirectional C++ ↔ Rust | `cxx` |
| Python extension that manipulates Python objects / GIL / numpy | `pyo3` (+ maturin) — BoltFFI only exposes Rust-shaped APIs |
| Node-native addon needing N-API specifics | `napi-rs` |
| Existing UniFFI codebase | Keep UniFFI unless FFI overhead is measured as a problem; then migrate to BoltFFI |

**Do NOT** hand-write `extern "C"` + `#[repr(C)]` + per-language glue when BoltFFI covers the targets.
That's where most FFI UB comes from (see `unsafe-checker` ffi-* rules).

## Setup

```bash
cargo install boltffi_cli     # binary is `boltffi`
cargo add boltffi
boltffi init                  # creates boltffi.toml
boltffi check --fix           # installs missing rustup targets
```

```toml
# Cargo.toml
[package]
edition = "2024"

[lib]
crate-type = ["cdylib", "staticlib"]

[dependencies]
boltffi = "0.32"   # default features: uuid, url. Optional: chrono, fast-alloc
```

```rust
// src/lib.rs — REQUIRED once per crate that declares BoltFFI items (since 0.32)
boltffi::scaffolding!();
```
Missing it → ``cannot find type `BoltffiScaffolding` in the crate root``.

## Build & Package

```bash
boltffi pack all --release            # → dist/apple, dist/android, dist/java, dist/csharp, dist/wasm, dist/python
boltffi pack apple --release          # target is `apple`, NOT `ios` (some docs pages are stale)
boltffi pack c --experimental --deny-skipped
boltffi generate swift -o out/        # bindings only
boltffi release                       # check → build → generate → pack
```
Use `--deny-skipped` in CI so an unsupported signature fails the build instead of silently disappearing.

## API Cheat Sheet (0.32)

| Rust | Annotation | Foreign side |
|------|-----------|--------------|
| Free function | `#[export] pub fn` | top-level fn / static method |
| Value type (struct / enum, copied) | `#[data]` | struct / data class / record / sealed type |
| Methods & constructors on a value type | `#[data(impl)] impl T` | `new` → constructor; other `-> Self` → named ctors |
| Stateful object (lives in Rust, handle across) | `#[export]` on **struct AND impl** | class |
| Error | `#[error]` enum/struct, return `Result<T, E>` | thrown error / exception |
| Async | `#[export] pub async fn` | async/await, coroutines, `Task`, Promise, asyncio |
| Callback interface | `#[export] pub trait` + `Box<dyn T>` / `Arc<dyn T>` | protocol / interface implemented by caller |
| Sync closure (call-scoped only) | `f: impl Fn(..) -> ..` param | lambda |
| Stream | `#[ffi_stream(item = T)]` → `Arc<EventSubscription<T>>` | AsyncStream / Flow / IAsyncEnumerable |
| Constant | `#[export] pub const` | constant |
| Foreign/remote type | `custom_type! { .. }` | mapped primitive |
| Field default | `#[boltffi::default(3)]` | default argument |

### Values, functions, errors

```rust
use boltffi::*;

boltffi::scaffolding!();

#[data]
#[derive(Clone, Copy)]
pub struct Point {
    pub x: f64,
    pub y: f64,
}

#[data(impl)]
impl Point {
    pub fn new(x: f64, y: f64) -> Self { Self { x, y } }
}

#[error]
pub enum ApiError {
    Network { message: String },
    NotFound,
    RateLimited { retry_after: u32 },
}

#[export]
pub fn distance(a: Point, b: Point) -> f64 {
    ((b.x - a.x).powi(2) + (b.y - a.y).powi(2)).sqrt()
}

#[export]
pub fn greet(name: &str) -> String { // &str in, String out
    format!("Hello, {name}!")
}
```

### Classes (thread-safe by default)

```rust
use std::sync::Mutex;

#[export]
pub struct Counter {
    value: Mutex<i32>, // fields stay private
}

#[export]
impl Counter {
    pub fn new() -> Self { Self { value: Mutex::new(0) } }
    pub fn increment(&self) { *self.value.lock().expect("counter mutex poisoned") += 1; }
    pub fn get(&self) -> i32 { *self.value.lock().expect("counter mutex poisoned") }
}
```
- Classes must be `Send + Sync`; methods take `&self` (interior mutability).
- `&mut self` requires `#[export(single_threaded)]` **on the struct** (not the impl — that was 0.31 syntax and is now an error). Only use it when access is truly confined to one thread.
- `#[skip]` hides a method. Export impl blocks cannot be generic or trait impls.

### Callbacks & async

```rust
#[export]
pub trait ProgressHandler: Send + Sync {
    fn on_progress(&self, percent: f64);
    fn on_complete(&self, result: &str);
}

#[export]
pub async fn download(url: &str, handler: Arc<dyn ProgressHandler>) -> Result<Vec<u8>, ApiError> {
    // foreign side drives polling — no runtime needed by BoltFFI itself,
    // BUT tokio-based libs (reqwest, tokio::fs) still need a tokio runtime you start.
    todo!()
}
```
`Box<dyn T>` for single-use, `Arc<dyn T>` when stored. Closures (`impl Fn`) must not outlive the call.

## Design Rules for FFI-Friendly APIs

| Rule | Why |
|------|-----|
| Keep hot-path `#[data]` types **primitive-only** | they cross as plain C structs (zero-copy); `String`/`Vec` fields force wire encoding |
| **Batch** string/collection work into one call | per-call encoding cost dominates small calls |
| Smallest integer that fits; `i64/u64` become `bigint` in TS | size + JS ergonomics |
| `&str` / `&[T]` params, owned returns | borrowed returns are unsupported |
| Concrete (monomorphic) types only | no generics, no `dyn Trait` values (use enum), no raw pointers, no `HashSet` (use `Vec`) |
| Export names unique **crate-wide** | not namespaced per module; `Duration`, `SystemTime`, `Uuid`, `Url` are reserved |
| Validate inputs at the API, not in `custom_type!` | a failed `try_from_ffi` **panics** |
| Streams drop events when buffer (default 256) is full | size the buffer or use backpressure in your design |
| Path-dependency crates with exports must be referenced: `use shapes as _;` | otherwise `generate` fails |
| Large items (≥64 types) need `#![recursion_limit = "256"]` | macro expansion depth |

## Payloads that leave the process

BoltFFI's internal wire format (LE primitives, `u32` length prefixes, no schema) is **only for the in-process
call boundary** — never persist it or send it over the network. When an FFI API carries a payload that is also
stored, sent over IPC/network, or shared with non-Rust services, pass it as `Vec<u8>` / `&[u8]`
(→ `Data` / `ByteArray` / `Uint8Array`) encoded with the project serialization priority:

**Cap'n Proto → Protobuf → JSON** (see `m11-ecosystem/serialization.md`).

## Related Skills

| When | See |
|------|-----|
| Hand-written `unsafe extern "C"`, `#[repr(C)]` | unsafe-checker |
| Choosing serialization formats | m11-ecosystem (`serialization.md`) |
| Async runtime inside the library | m07-concurrency |
| Error type design for `#[error]` | m06-error-handling |
