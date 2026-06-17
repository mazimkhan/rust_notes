# 04 — Functions & Control Flow

## Concept summary

### Functions

- Defined with `fn`; parameters need types; return type after `->`.
  ```rust
  fn add(a: i32, b: i32) -> i32 {
      a + b   // no semicolon: this is an *expression* and the return value
  }
  ```
- **Statements** end with `;` and produce no value. **Expressions** evaluate to a value. Most things in Rust are expressions — including `if`, `match`, and block `{ ... }`.
- Early return: `return value;` (with `;`).
- Unit return `()` is the default if `-> T` is omitted.

### Control flow

```rust
let x = if cond { 1 } else { 2 };     // if is an expression
```

- `if` requires a `bool` (no implicit conversion from ints).
- All branches of an expression `if`/`match` must have the **same type**.

Loops:

```rust
loop { ... }                          // infinite; break with `break value;`
while cond { ... }
for x in 0..10 { ... }                // range, exclusive end
for x in 0..=10 { ... }               // inclusive end
for item in vec.iter() { ... }
```

- `loop { break 42; }` returns a value from a labeled loop.
- Nested loops can use labels: `'outer: loop { ... break 'outer; }`.
- Prefer `for` over manual indexing — it's safer and faster.

## Quiz

1. Why does removing the semicolon after `a + b` matter in the `add` function above?
2. Is this valid? Why or why not?
   ```rust
   let n = if 1 { "yes" } else { "no" };
   ```
3. Predict the value of `x`:
   ```rust
   let x = loop {
       break 7 * 6;
   };
   ```
4. Rewrite this `while` loop as a `for` loop:
   ```rust
   let mut i = 0;
   while i < 5 {
       println!("{i}");
       i += 1;
   }
   ```
5. What's wrong with this code?
   ```rust
   fn classify(n: i32) -> &str {
       if n > 0 { "positive" } else if n < 0 { "negative" }
   }
   ```
6. Write a function `is_even(n: i32) -> bool` using an `if` expression as the body (no `return`).

<details>
<summary>Answers</summary>

1. With `;` it becomes a statement, which evaluates to `()`. The function would then need an explicit `return` or its declared return type would mismatch. Without `;` the line is the trailing expression and its value is returned.
2. **Invalid.** `if` requires a `bool` — Rust does not coerce integers to booleans. Use `if 1 != 0` or refactor.
3. `42` — `break value` inside `loop` returns that value from the loop expression.
4. ```rust
   for i in 0..5 {
       println!("{i}");
   }
   ```
5. (a) The return type should be `&'static str` (a lifetime is needed for borrowed `str`) — or just `String`. (b) More importantly, not all paths return a value: the missing `else` branch makes the expression evaluate to `()`, which mismatches the declared return type. Add `else { "zero" }`.
6. ```rust
   fn is_even(n: i32) -> bool {
       if n % 2 == 0 { true } else { false }
       // idiomatic: n % 2 == 0
   }
   ```
</details>
