---
name: rust-assert-context
description: "Rust (Google style): give `assert!` a context message explaining why the condition holds, the offending value, or who misused the API."
# TTSR: Google Rust Style Guide, "Use context strings to clarify assertions".
# Condition: `assert!`/`debug_assert!` with a single argument (no top-level
# comma; nested calls up to two levels deep are understood).
condition:
  - '\b(?:debug_)?assert!\((?:[^(),]|\((?:[^()]|\([^()]*\))*\))*\)'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: assertion messages (Google Rust Style Guide)

`assert!` and `debug_assert!` accept a format message. Add one unless the condition explains itself. Unlike a comment, the message appears in logs and panic output.

```rust
// Why the condition should hold:
assert!(values.is_sorted(), "values come from a PriorityQueue, so they must be sorted");
// The offending value:
assert!(refund > 0, "refund must be positive, got {refund:?}");
// Who is at fault (caller misuse vs. internal bug):
assert!(self.stream.is_some(), "call connect() before execute_query()");
```

Do not just restate the condition: `"values should be sorted"` adds nothing.

If the condition is trivially obvious, or a test assertion's failure output already explains it, move on.
