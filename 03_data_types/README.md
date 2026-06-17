# 03 — Data Types

## Concept summary

Rust is **statically and strongly typed**. The compiler infers most types, but you must annotate when ambiguous (e.g. `parse`).

### Scalars

| Category | Examples | Notes |
|---|---|---|
| Signed ints | `i8 i16 i32 i64 i128 isize` | `i32` is the default |
| Unsigned ints | `u8 u16 u32 u64 u128 usize` | `usize` indexes memory |
| Floats | `f32 f64` | `f64` is the default |
| Booleans | `bool` | only `true` / `false` |
| Characters | `char` | 4-byte Unicode scalar, single quotes `'a'` |

- Integer literals can have a type suffix and `_` separators: `1_000_000_u32`.
- Integer overflow: **panics in debug, wraps in release** (configurable). Use `checked_*`, `wrapping_*`, `saturating_*`, `overflowing_*` for explicit behavior.

### Compound types

- **Tuple** — fixed length, possibly heterogeneous: `let t: (i32, f64, char) = (1, 2.0, 'a');`
  - Access by index: `t.0`, `t.1`. Or destructure: `let (a, b, c) = t;`
  - The empty tuple `()` is the **unit** type — Rust's "no value".
- **Array** — fixed length, all same type, stack-allocated: `let a: [i32; 5] = [1, 2, 3, 4, 5];`
  - Default-fill: `let zeros = [0; 10];` — 10 zeros.
  - Bounds-checked at runtime; out-of-range index → panic.
- For dynamic-length lists use **`Vec<T>`** (covered in module 09).

### Strings (preview)

- `&str` — borrowed string slice (often a string literal): `"hi"`.
- `String` — owned, growable, heap-allocated.

## Quiz

1. What is the default integer type? The default float type?
2. What does `usize` represent and when do you use it?
3. Predict — does this compile?
   ```rust
   let x: u8 = 255;
   let y = x + 1;
   ```
   What happens at runtime in debug vs. release?
4. Destructure this tuple and print only the second element:
   ```rust
   let point = (3, 4.5, "label");
   ```
5. What's the difference between `[0; 5]` and `[0, 5]`?
6. Which of these is a `char` literal: `"a"`, `'a'`, `b'a'`?

<details>
<summary>Answers</summary>

1. Default integer is `i32`; default float is `f64`.
2. `usize` is an unsigned integer the size of a pointer on the target platform (32-bit on 32-bit machines, 64-bit on 64-bit). Use it for **indexing, lengths, capacities**.
3. It compiles fine. At runtime: **debug → panic** ("attempt to add with overflow"); **release → wraps to 0** silently (unless overflow checks are enabled).
4. ```rust
   let (_, y, _) = point;
   println!("{y}");
   ```
5. `[0; 5]` is an array of five zeros (`[0, 0, 0, 0, 0]`). `[0, 5]` is an array of two elements: `0` and `5`.
6. `'a'` is the `char`. `"a"` is `&str`. `b'a'` is a `u8` byte literal.
</details>
