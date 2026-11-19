---
name: rust-api-design
description: "Rust (Google style): API shape: enum over bool params, sparing `impl Into<T>`, generics over `dyn`, named fields over tuples, iterators over `Vec`, Option/Result/panic by contract."
# TTSR: Google Rust Style Guide, "Apply API design rules in context",
# "Function inputs and outputs", "Types".
# Conditions: a `pub fn` with a `bool` parameter; an `impl Into<…>` parameter;
# a `pub fn` taking `&dyn`/`Box<dyn`; a `pub fn` whose return type (right
# after the parameter list) is a 2+ tuple, bare or in Option/Result; a tuple
# struct with 2+ top-level fields.
condition:
  - '\bpub(?:\([^)\n]*\))?[ \t]+(?:(?:const|async|unsafe|extern(?:[ \t]+"[^"]*")?)[ \t]+)*fn[ \t]+\w+[^(]*\([^)]*\b\w+[ \t]*:[ \t]*bool\b'
  - '\w[ \t]*:[ \t]*impl[ \t]+Into[ \t]*<'
  - '\bpub(?:\([^)\n]*\))?[ \t]+(?:(?:const|async|unsafe|extern(?:[ \t]+"[^"]*")?)[ \t]+)*fn[ \t]+\w+[^(]*\([^)]*:[ \t]*(?:&(?:''\w+[ \t]+)?(?:mut[ \t]+)?|Box[ \t]*<[ \t]*)dyn\b'
  - '\bpub(?:\([^)\n]*\))?[ \t]+(?:(?:const|async|unsafe|extern(?:[ \t]+"[^"]*")?)[ \t]+)*fn[ \t]+\w+[^(]*\((?:[^()]|\([^()]*\))*\)[ \t]*->[ \t]*(?:Option[ \t]*<[ \t]*|(?:[\w:]*::)?Result[ \t]*<[ \t]*)?\([^()]*,[^()]*\)'
  - '\bstruct[ \t]+\w+(?:<[^>]*>)?[ \t]*\((?:[^(),<>;]|<(?:[^<>]|<(?:[^<>]|<[^<>]*>)*>)*>)*,[ \t\r\n]*[^\s);]'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: API design (Google Rust Style Guide)

These rules are binding for library code, meaning code other projects use. For application code they are a strong recommendation, but simplicity beats reusability there, so do not generalize prematurely. Where this guide is silent, follow the Rust API Guidelines (rust-lang.github.io/api-guidelines).

- **Use an `enum` instead of a `bool` parameter** when the meaning is not obvious at the call site: write `open(path, Mode::ReadOnly)`, not `open(path, true)`. If a `bool` stays, annotate it at the call: `f(/* recursive= */ true)`.
- **Take `impl Into<T>` only when call-site ergonomics are paramount** in a library API, and weigh it against code bloat. Application code should almost never take `impl Into<T>`.
- **Prefer generics to `dyn Trait` parameters** in libraries (`T: Trait` or `impl Trait`): no vtable indirection, and calls can be inlined. Use `dyn` when code size matters.
- **Prefer named fields to tuples** and to tuple structs with more than one field, because `.0` and `.1` quickly become unreadable. One-field newtypes and very local tuples are fine.
- **Use a newtype instead of a type alias** when the type has its own meaning or API, may change representation, or is easy to mix up with similar values (`Day`, `Month`, `Year`).
- **Return iterators** (`impl Iterator<Item = T>`) rather than `Vec<T>`, and accept `impl IntoIterator` rather than `&[T]` when the function only iterates. Exceptions: most callers need a `Vec`; the implementation builds one anyway; the result will be indexed; the algorithm reads better as a `for` loop and the allocation barely matters; producing a collection is the function's purpose; the function returns an existing vector. Application code may simply use `Vec` and slices.
- **Choose the error channel by contract**:
  - `Option` when the cause of failure is obvious (a missing value, an overflow).
  - `Result` when the caller needs to know why (I/O, network, parse errors).
  - `panic!` only for bugs and violated preconditions, documented under `# Panics`. Tests and developer-facing binaries may panic on non-fatal errors; some codebases ban panics entirely and report broken invariants through `Result`.

If the code is application or internal code where the simpler signature reads better, or the signature comes from a trait or FFI binding you do not control, move on.
