---
name: rust-string-format
description: "Rust (Google style): `format!`/`write!` for joining more than two strings and for substitution into literals; no `+` chains or `.replace(\"{x}\", …)` templating."
# TTSR: Google Rust Style Guide, "Prefer format! / write! for concatenation
# and substitution".
# Conditions: a `+` chain of three or more string pieces; `.replace()` on a
# string literal or with a placeholder-looking pattern (`{name}`, `$NAME`,
# `%s`, `<NAME>`, `__NAME__`).
condition:
  - '\+[ \t]*"[^"\n]*"[ \t]*\+'
  - '\+[ \t]*&[\w.]+(?:\(\))?[ \t]*\+[ \t]*"'
  - '\.to_(?:string|owned)\(\)[ \t]*\+[ \t]*&?[\w.]+(?:\([^()]*\))?[ \t]*\+'
  - '"[^"\n]*"[ \t]*\.replacen?\('
  - '\.replacen?\([ \t]*"(?:\{\{?\w+\}?\}|\$\{?\w+\}?|%\w|<\w+>|__\w+__)"'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: building strings (Google Rust Style Guide)

- **Join more than two pieces with `format!("{a}{b}{c}")`**, or with `write!(buf, …)` into an existing `String` or writer, rather than `a + &b + &c`. Each `+` step can reallocate.
- **Substitute values into literal text with a format string** (`format!("Hello {name}")`), not `"Hello NAME".replace("NAME", name)` or a home-made `{placeholder}`/`$VAR` template syntax.
- Two-piece joins (`a + &b`, `s.push_str(x)`) and `.join(sep)` over a collection are fine.

If the template is loaded at runtime from a file or config rather than written as a literal in the code, move on.
