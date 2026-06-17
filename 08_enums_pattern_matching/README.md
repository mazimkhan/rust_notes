# 08 — Enums & Pattern Matching

## Concept summary

Enums are *sum types*: a value is one of several **variants**, each of which may carry data.

```rust
enum Shape {
    Circle(f64),                       // tuple-like variant
    Rectangle { w: f64, h: f64 },      // struct-like variant
    Unit,                              // no payload
}
```

### `Option<T>` & `Result<T, E>`

Built-in enums you'll use constantly:

```rust
enum Option<T> { None, Some(T) }
enum Result<T, E> { Ok(T), Err(E) }
```

Rust has no `null`. Absence is modeled with `Option`.

### `match`

A match is an **expression**; it must be **exhaustive** (cover every case):

```rust
fn area(s: Shape) -> f64 {
    match s {
        Shape::Circle(r) => std::f64::consts::PI * r * r,
        Shape::Rectangle { w, h } => w * h,
        Shape::Unit => 0.0,
    }
}
```

- `_` is the wildcard catch-all.
- Match guards: `Some(n) if n > 0 => ...`.
- Binding with `@`: `n @ 1..=9 => ...`.
- Patterns work in `if let`, `while let`, `let`, function parameters, and `for`.

### Concise alternatives

```rust
if let Some(x) = maybe { use_it(x); }
while let Some(top) = stack.pop() { /* ... */ }
let Some(x) = maybe else { return; };       // let-else, requires the rest to diverge
```

### Common `Option` / `Result` combinators

`unwrap`, `expect`, `unwrap_or`, `unwrap_or_else`, `map`, `and_then`, `ok_or`, `is_some`, `?` (next module).

## Quiz

1. Why does this fail to compile?
   ```rust
   match n {
       1 => "one",
       2 => "two",
   }
   ```
2. Rewrite using `if let`:
   ```rust
   match maybe {
       Some(x) => println!("{x}"),
       None => {}
   }
   ```
3. What's the difference between `unwrap()` and `expect("msg")`?
4. Predict the output:
   ```rust
   let x: Option<i32> = Some(5);
   let y = x.map(|n| n * 2).unwrap_or(0);
   println!("{y}");
   ```
5. Design an enum `Event` with variants `Click { x: i32, y: i32 }`, `KeyPress(char)`, and `Close`. Write a `match` that prints something for each.
6. What does `let-else` give you that plain `if let` doesn't?

<details>
<summary>Answers</summary>

1. `match` must be exhaustive. Missing pattern for other `i32` values. Add `_ => "other"` (and likely a return type).
2. ```rust
   if let Some(x) = maybe { println!("{x}"); }
   ```
3. Both panic on `None`/`Err`. `expect("msg")` adds a custom panic message — preferred for debuggability.
4. `10`
5. ```rust
   enum Event { Click { x: i32, y: i32 }, KeyPress(char), Close }

   match ev {
       Event::Click { x, y } => println!("click at ({x},{y})"),
       Event::KeyPress(c) => println!("key {c}"),
       Event::Close => println!("close"),
   }
   ```
6. `let-else` binds inside the *surrounding* scope on the `Some(_)`/`Ok(_)` path and diverges (returns/breaks/panics) on the failure path. With `if let` the bound name is only in scope inside the `if` branch, leading to "rightward drift".
</details>
