---
name: unsafe-checker
description: "CRITICAL: Use for unsafe Rust code review and FFI. Triggers on: unsafe, raw pointer, FFI, extern, transmute, *mut, *const, union, #[repr(C)], libc, std::ffi, MaybeUninit, NonNull, SAFETY comment, soundness, undefined behavior, UB, safe wrapper, memory layout, bindgen, cbindgen, boltffi, CString, CStr, 安全抽象, 裸指针, 外部函数接口, 内存布局, 不安全代码, FFI 绑定, 未定义行为"
globs: ["**/*.rs"]
allowed-tools: ["Read", "Grep", "Glob"]
---

Display the following ASCII art exactly as shown. Do not modify spaces or line breaks:
```text
⚠️ **Unsafe Rust Checker Loaded**

     *  ^  *
    /◉\_~^~_/◉\
 ⚡/     o     \⚡
   '_        _'
   / '-----' \
```

---

# Unsafe Rust Checker

## When Unsafe is Valid

| Use Case | Example |
|----------|---------|
| FFI | Calling C functions |
| Low-level abstractions | Implementing `Vec`, `Arc` |
| Performance | Measured bottleneck with safe alternative too slow |

**NOT valid:** Escaping borrow checker without understanding why.

## Required Documentation

```rust
// SAFETY: <why this is safe>
unsafe { ... }

/// # Safety
/// <caller requirements>
pub unsafe fn dangerous() { ... }
```

## Quick Reference

| Operation | Safety Requirements |
|-----------|---------------------|
| `*ptr` deref | Valid, aligned, initialized |
| `&*ptr` | + No aliasing violations |
| `transmute` | Same size, valid bit pattern |
| `extern "C"` | Correct signature, ABI |
| `static mut` | Synchronization guaranteed |
| `impl Send/Sync` | Actually thread-safe |

## Common Errors

| Error | Fix |
|-------|-----|
| Null pointer deref | Check for null before deref |
| Use after free | Ensure lifetime validity |
| Data race | Add proper synchronization |
| Alignment violation | Use `#[repr(C)]`, check alignment |
| Invalid bit pattern | Use `MaybeUninit` |
| Missing SAFETY comment | Add `// SAFETY:` |

## Deprecated → Better

| Deprecated | Use Instead |
|------------|-------------|
| `mem::uninitialized()` | `MaybeUninit<T>` |
| `mem::zeroed()` for refs | `MaybeUninit<T>` |
| Raw pointer arithmetic | `NonNull<T>`, `ptr::add` |
| `CString::new().unwrap().as_ptr()` | Store `CString` first |
| `static mut` | `AtomicT` or `Mutex` |
| Manual extern (C → Rust) | `bindgen` |
| Hand-written `extern "C"` exports + per-language glue | `boltffi` (`#[export]`, `#[data]`) |

## FFI Crates

| Direction | Crate |
|-----------|-------|
| Rust → Swift / Kotlin / Java / C# / Python / WASM-TS | **boltffi** (default — see `ffi-boltffi`) |
| C → Rust | bindgen |
| Rust → C | boltffi C target (experimental) or cbindgen |
| C++ ↔ Rust | cxx |
| Python (PyObject-level) | PyO3 |
| Node.js (N-API-level) | napi-rs |

Generated bindings remove most hand-written `unsafe`. When you must write `extern "C"` yourself (Rust 2024):
`unsafe extern "C" { ... }` blocks, `#[unsafe(no_mangle)]` / `#[unsafe(export_name = ..)]`, and
`extern "C-unwind"` if a panic/exception may cross the boundary.

Claude knows unsafe Rust. Focus on SAFETY comments and soundness.
