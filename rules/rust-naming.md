---
name: rust-naming
description: "Rust (Google style): std-style accessors (no `get_`), as_/to_/into_/from_ self conventions, `try_new` for fallible constructors, descriptive type lifetimes, `Ext` extension traits."
# TTSR: Google Rust Style Guide, "Naming" and "Getters and setters".
# Conditions: zero-arg `get_*` getter; `new(..) -> Result`; wrong self kind
# for `into_*`/`to_*`/`from_*`; single-letter lifetime on a type
# definition; three single-letter lifetime params; a non-`Ext` local trait
# implemented on a std type.
condition:
  - '\bfn[ \t]+get_(?!(?:mut|ref|pin_mut|pin_ref)\b)\w+[ \t]*(?:<[^>]*>)?[ \t]*\([ \t]*&(?:mut[ \t]+)?self[ \t]*\)'
  - '\bfn[ \t]+new[ \t]*(?:<[^>]*>)?[ \t]*\([^)]*\)[ \t]*->[ \t]*(?:[\w:]*::)?Result\b'
  - '\bfn[ \t]+into_\w+[ \t]*(?:<[^>]*>)?[ \t]*\([ \t]*&'
  - '\bfn[ \t]+to_(?!(?:\w*_)?mut\b)\w+[ \t]*(?:<[^>]*>)?[ \t]*\([ \t]*&mut[ \t]+self\b'
  - '\bfn[ \t]+from_\w+[ \t]*(?:<[^>]*>)?[ \t]*\([ \t]*&?(?:mut[ \t]+)?self\b'
  - '\b(?:struct|enum|union|trait)[ \t]+\w+[ \t]*<[^>{]*''[a-z]\b(?!'')'
  - '''[a-z][ \t]*,[ \t]*''[a-z][ \t]*,[ \t]*''[a-z]\b(?!'')'
  - '\bimpl(?:<[^>]*>)?[ \t]+[A-Z]\w*(?<!Ext)[ \t]+for[ \t]+&?(?:''\w+[ \t]+)?(?:str|String|Vec|Option|Result|HashMap|HashSet|BTreeMap|BTreeSet|VecDeque|Path|PathBuf|Duration|Instant|SystemTime|u8|u16|u32|u64|u128|usize|i8|i16|i32|i64|i128|isize|f32|f64|bool|char)\b'
scope:
  - tool:edit(*.rs)
  - tool:write(*.rs)
---

# Rust: naming (Google Rust Style Guide)

- **Accessors** (no `get_` prefix):
  - `foo()` returns `&T`, or `T` when `T: Copy`.
  - `foo_mut()` returns `&mut T`.
  - `set_foo(v)` sets the value.
  - `take_foo()` leaves `None` or a default in place and returns the old value.
  - `replace_foo(v)` sets the value and returns the old one.
  - `with_foo(|f| …)` and `with_mut_foo(|f| …)` give closure access: to state behind a `Mutex`/`RefCell` (instead of leaking the guard), or when a change needs follow-up work or validation and a `set_foo()` will not do.
- **Conversions follow std** (clippy `wrong_self_convention`):
  - `as_*(&self)` or `as_*(&mut self)`: cheap view, borrow to borrow.
  - `to_*(&self)`, or `self` for `Copy` types: produces a new, possibly expensive value. `to_*_mut(&mut self)` for the mutable variant.
  - `into_*(self)`: consumes the receiver.
  - `from_*`: an associated function with no `self`.
- **Constructors**: `new()`, plus `new_with_foo()` when there are several. A constructor that returns `Result` is named `try_new()`. Value-to-value conversions are `from_foo()`, paired with `impl From<Foo>`.
- **Lifetimes**: give type lifetimes descriptive names (`struct Parser<'src>`, not `<'a>`), except in very generic abstractions such as `Cow`. Use `'_` wherever it works. Single letters are fine for tying a method's inputs to its output and for local `for<'a>` bounds, at most two of them; beyond that, name them.
- **Generic parameters**: short names that hint at the main bound (`R: Read`, `K`, `V`). Use `T`/`U` only for nearly unbounded generics such as `Vec<T>`.
- **Extension traits**, meaning local traits that only add methods to a foreign type, end in `Ext` (`StrExt`, `ForeignTypeExt`). The team that owns the extended type should own its extension traits, except for widely useful extensions in foundational libraries. Input traits implemented on many foreign types (`DateLike`, codecs) keep plain names.
- **No obscure abbreviations or dropped letters** (`cnt`, `mgr`, `svc`). Abbreviations common enough to appear in Wikipedia (`Ip`, `Http`, `Url`) are fine.
- In every other case, method names mirror the standard library (std-style wrapper accessors such as `get_mut()`/`get_ref()` are fine).

If the name is fixed by a trait you implement, an external API or FFI binding, or a convention used across the repo, move on.
