---
name: rust-unsafe
description: "Rust (Google style): unsafe only when no safe way exists, every block with a `// SAFETY:` comment, every unsafe fn/trait with `# Safety` docs, wrapped in a small safe abstraction."
# TTSR: Google Rust Style Guide, "Unsafe code".
# Condition: the written snapshot has an `unsafe` block/fn/impl/trait/extern
# but no `SAFETY:` comment and no `# Safety` doc section anywhere in it.
condition:
  - '^(?![\s\S]*(?:SAFETY:|#[ \t]*Safety\b))[\s\S]*?\bunsafe[ \t]*(?:\{|fn[ \t]+\w|impl\b|trait\b|extern[ \t]*(?:"[^"]*"[ \t]*)?(?:\{|fn[ \t]+\w))'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: unsafe code (Google Rust Style Guide)

You added unsafe code without a `SAFETY:` comment or a `# Safety` section.

- **Write `unsafe` only when no safe implementation exists.** If the reason is performance, there must be benchmarks.
- **Precede every `unsafe { … }` block and `unsafe impl` with a `// SAFETY:` comment** explaining why each precondition holds at that spot.
- **Document caller obligations** for every `unsafe fn` and `unsafe trait` under `/// # Safety`.
- **Record the justification** (why safe code will not do, and the benchmarks if the reason is performance) once per block, file, or build target, whichever fits.
- **Wrap unsafe code in a small safe abstraction** kept apart from business logic, ideally in its own crate with few dependencies beyond std.

If the `SAFETY:` comment or `# Safety` section already exists just outside the lines you edited, move on.
