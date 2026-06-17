# 05 — Ownership & Move Semantics

> Ownership is Rust's killer feature: it gives memory safety **without a garbage collector**.

## Concept summary

### The three rules

1. Every value has a single **owner** (a variable binding).
2. There can only be one owner **at a time**.
3. When the owner goes out of scope, the value is **dropped** (memory freed).

### Move vs. Copy

- Types stored *entirely on the stack* with a known fixed size and no destructor implement the `Copy` trait (`i32`, `bool`, `char`, `f64`, fixed-size arrays of `Copy` types, tuples of `Copy` types). Assignment **copies**.
- Heap-owning types (`String`, `Vec<T>`, `Box<T>`, etc.) are **not** `Copy`. Assignment or passing to a function **moves** ownership — the original binding is *invalidated*.

```rust
let s1 = String::from("hi");
let s2 = s1;          // move; s1 is now invalid
println!("{s1}");     // compile error: value borrowed after move
```

- Use `.clone()` for a deep copy when you really need both.

### Functions

- Passing a non-`Copy` value moves it into the function. To keep it usable afterward, either:
  - pass a **reference** (`&` or `&mut`) — covered in module 06, or
  - have the function return ownership back.

### Drop

- `Drop::drop` runs deterministically when a value goes out of scope (RAII). Custom destructors via `impl Drop for MyType`.

## Quiz

1. Does this compile?
   ```rust
   let s1 = String::from("hi");
   let s2 = s1;
   println!("{s1}");
   ```
2. Same question for:
   ```rust
   let x = 42;
   let y = x;
   println!("{x} {y}");
   ```
   Why does it differ from question 1?
3. What gets printed?
   ```rust
   fn takes(s: String) -> String { s }
   let s = String::from("yo");
   let s = takes(s);
   println!("{s}");
   ```
4. Add the smallest change to make this compile (without using references):
   ```rust
   fn yell(s: String) { println!("{}!", s); }
   let s = String::from("hi");
   yell(s);
   yell(s);
   ```
5. When does `Drop::drop` run for a local `String`?
6. Which is `Copy`? `i32`, `String`, `&str`, `Vec<i32>`, `(i32, bool)`, `(i32, String)`.

<details>
<summary>Answers</summary>

1. **No.** `String` is not `Copy`; `let s2 = s1;` moves out of `s1`, which can no longer be used.
2. **Yes, it compiles.** `i32` is `Copy`, so `let y = x;` copies the bits. Both `x` and `y` are valid.
3. `yo` — the function takes ownership and returns it; we rebind to `s` via shadowing.
4. Clone before the first call (or before the second): `yell(s.clone()); yell(s);`. The idiomatic fix is to take `&str`/`&String`, but that uses references.
5. When the binding goes out of scope (end of block), the compiler inserts a call to `Drop::drop` which frees the heap buffer.
6. `Copy`: `i32`, `&str` (it's a reference, which is `Copy`), `(i32, bool)`. **Not** `Copy`: `String`, `Vec<i32>`, `(i32, String)` (tuple with a non-`Copy` field).
</details>
