---
name: rust-borrowed-params
description: "Rust (Google style): borrow the data, not its container: `&str`/`&[T]`/`&T`/`Option<&T>` instead of `&String`/`&Vec<T>`/`&Box<T>`/`&Option<T>`."
# TTSR: Google Rust Style Guide, "Accept flexible references for immutable data".
# Condition: a shared reference to an owning container right after a named
# parameter/field/binding (`name: &String`) or a return arrow (`-> &Vec<T>`);
# generic args like `Vec<(&String, &u32)>` stay quiet. `&mut` and
# expressions like `&String::from(..)` do not match.
condition:
  - '(?:\w[ \t]*:|->)[ \t]*&(?:''\w+[ \t]+)?(?:(?:Vec|Box|Option)[ \t]*<|(?:String|PathBuf|OsString)\b(?!::))'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: borrow the data, not the container (Google Rust Style Guide)

When a function borrows immutable data, take a reference to the data itself rather than to a smart pointer, container, or wrapper. Callers can then pass data held in any form without copying it first (clippy `ptr_arg`, `borrowed_box`).

| Use | Instead of |
|---|---|
| `&T` | `&Box<T>` |
| `&[T]` | `&Vec<T>` |
| `&str` | `&String` |
| `Option<&T>` (callers pass `opt.as_ref()`) | `&Option<T>` |
| `&Path` / `&OsStr` | `&PathBuf` / `&OsString` |

`&mut Vec<T>` and `&mut String` are fine when the function grows or shrinks the container. Also fine: code that needs the container's own API (`capacity()`, non-linear text such as ropes or cords), and a trait or FFI signature you do not control. In those cases, move on.
