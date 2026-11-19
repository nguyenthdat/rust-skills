---
name: rust-iterator-chains
description: "Rust (Google style): `for` loops over long or side-effecting combinator chains; no chains in a `for` header; avoid `for_each`."
# TTSR: Google Rust Style Guide, "Don't use iterator combinator chains when
# direct iteration is clearer".
# Conditions: `.for_each(`; a closure-taking adapter inside a `for … in`
# header; an adapter closure that mutates outside state (`+=`, `.push(`, …).
condition:
  - '\.for_each\('
  - '\bfor[ \t]+[^{;\n]+?[ \t]+in[ \t\r\n]+[^{;]*?\.\w+\([ \t\r\n]*(?:move[ \t]+)?\|'
  - '\.(?:map|filter|filter_map|flat_map|inspect|take_while|skip_while|any|all|find)\([ \t]*(?:move[ \t]+)?\|[^|]*\|[ \t]*\{(?![^}]*\blet[ \t]+mut\b)[^}]*?(?:\+=|-=|\*=|\.push\(|\.insert\()'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: iterator chains vs. loops (Google Rust Style Guide)

- **Keep combinator chains short and free of side effects.** A chain that branches, mutates outside state (`count += 1` or `out.push(…)` inside `map`/`filter`), or runs many steps reads better as a `for` loop with a `match`.
- **Never build a chain in a `for` header.** Bind it to a descriptive variable first: `let urgent_bugs = bugs.iter().filter(…);`, then `for bug in urgent_bugs { … }`.
- **Avoid `Iterator::for_each`** without a concrete reason. A `for` loop is easier to read and supports `?`, `break`, and `continue`.
- **Do use combinators** when the adapter itself carries the logic: `values.iter().any(|&v| v > threshold)` beats a hand-written loop with a flag and a `break`.

If the chain is short and its adapter carries the logic, move on.
