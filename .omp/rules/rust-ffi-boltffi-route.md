---
name: rust-ffi-boltffi-route
description: "Routes planning of Rust bindings for Swift/Kotlin/Java/C#/Python/WASM to BoltFFI before any code is written."
# TTSR prose router for `rust-ffi-boltffi`: fires while the model plans in
# text/thinking, before edits. prose-only, like the kotlin-* routers: a match
# in reasoning interrupts early, a match in tool args only leaves a reminder.
# Conditions: UniFFI/cbindgen named as the plan, or "Rust" near
# binding/FFI/SDK wording and a target language within one sentence.
condition:
  - '(?i)\b(?:use|using|add|adding|with|via|pick|choose)[ \t]+(?:uniffi|cbindgen)\b'
  - '(?i)\brust\b[^.\n]{0,60}\b(?:bindings?|ffi|sdk|export(?:ed|ing)?)\b[^.\n]{0,60}\b(?:swift|kotlin|android|ios|java|jvm|c#|\.net|python|typescript|wasm)\b'
scope: [text, thinking]
agents: [main, task, planner]
interruptMode: prose-only
---

# Rust bindings: plan with BoltFFI

This work exposes Rust to another language. The project default for Swift, Kotlin/Android, Java, C#, Python, and WASM/TypeScript is **BoltFFI** (`#[export]`, `#[data]`, `#[error]`, `boltffi pack`). If the rust-skills catalog is available, read `skill://ffi-boltffi` before you design the API.

Use something else only for C → Rust (`bindgen`), C++ (`cxx`), Python-object-level extensions (`pyo3`), DOM glue (`wasm-bindgen`), or an existing UniFFI codebase. If the bindings are only mentioned in passing and are not in scope, ignore this and continue.
