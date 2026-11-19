---
name: rust-ownership
description: "Rust (Google style): unique ownership and borrows over `Rc`/`Arc`; no `.clone()` as a substitute for borrowing."
# TTSR: Google Rust Style Guide, "Ownership and borrowing".
# Conditions: an `Rc`/`Arc` type or constructor; a clone that is only
# borrowed afterwards (`&x.clone()`, `x.clone().as_str()`, `.to_owned().clone()`).
condition:
  - '\b(?:Rc|Arc)(?:[ \t]*<|::new\()'
  - '&[\w.]+\.clone\(\)'
  - '\.clone\(\)\.(?:as_str|as_slice|as_bytes|as_path|as_ref|as_deref|iter|len|is_empty|get|contains|starts_with|ends_with)\('
  - '\.(?:clone|to_owned|to_string|to_vec)\(\)\.clone\(\)'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: ownership and borrowing (Google Rust Style Guide)

- **Avoid `Rc` and `Arc`.** Prefer one owner plus borrows: shared-ownership pointers add runtime cost and their own complexity, and engineers coming from other languages tend to reach for them too quickly. Use them only when the alternative ownership and borrowing structure would be unreasonably complex to write, understand, or maintain; that is a judgment call for code review. Once you have an `Rc`/`Arc`, cloning it is cheap and expected.
- **Do not `.clone()` instead of borrowing.** `&x.clone()`, `x.clone().as_str()`, and `x.clone().len()` copy data only to borrow it; borrow `&x` instead. Clone when the code needs its own deep copy, or as a pragmatic way out when the borrow relationships become excessively complex. In hot paths, weigh the allocation and copy.

If the alternative to shared ownership would be unreasonably complex, or the clone is a value the code keeps or moves, move on.
