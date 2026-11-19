---
name: rust-formatting
description: "Rust (Google style): rustfmt output at 100 columns, lowercase hex, escaped invisible characters, `#[rustfmt::skip]` only for alignment or generated code."
# TTSR: Google Rust Style Guide, "Code formatting and aesthetics".
# Conditions: `rustfmt::skip`, an uppercase hex literal, a raw
# zero-width/bidi/BOM/soft-hyphen character.
condition:
  - 'rustfmt::skip'
  - '(?<!")\b0x[0-9a-f_]*[A-F]'
  - '[\u00AD\u200B-\u200F\u202A-\u202E\u2060-\u2069\uFEFF]'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: formatting (Google Rust Style Guide)

- **Format with the repo's tool** before you finish: `cargo fmt`, or the project's wrapper such as `git cl format`. Do not format by hand.
- **Target settings**: `max_width = 100`, `hex_literal_case = "Lower"` (`0xab12`, not `0xAB12`), `format_macro_matchers = true`.
- **No raw invisible characters in literals**: zero-width joiners, bidi controls, soft hyphens, and BOMs are written as escapes (`"\u{200D}"`).
- **`#[rustfmt::skip]` is for two cases only**: deliberate manual alignment (tables, matrices, rows of similar lines kept aligned for readability) and generated code. It is never for personal style.

If the repo's `rustfmt.toml` sets something else (for example `max_width = 120`), or the file already writes hex in uppercase, follow the repo and move on.
