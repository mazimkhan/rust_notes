# 10 — Error Handling

## Concept summary

Rust splits errors into two categories:

- **Unrecoverable** → `panic!("…")`. Aborts (or unwinds) the thread. Use sparingly: bugs, invariant violations, prototypes.
- **Recoverable** → `Result<T, E>` — *the* idiomatic error type.

```rust
enum Result<T, E> { Ok(T), Err(E) }
```

### The `?` operator

`?` unwraps `Ok`/`Some` or **early-returns** the `Err`/`None`. It also calls `From::from` on the error, so you can return a unified error type from many sources.

```rust
use std::{fs, io};

fn read_first_line(path: &str) -> Result<String, io::Error> {
    let s = fs::read_to_string(path)?;          // io::Error propagates
    Ok(s.lines().next().unwrap_or("").to_string())
}
```

Works with `Option<T>` too (`None` short-circuits).

### Idiomatic combinators

| Goal | Use |
|---|---|
| Default on `None`/`Err` | `unwrap_or`, `unwrap_or_else`, `unwrap_or_default` |
| Transform success | `map` |
| Transform error | `map_err` |
| Chain fallible step | `and_then` |
| Inspect (no transform) | `inspect`, `inspect_err` |
| Convert `Option` to `Result` | `ok_or`, `ok_or_else` |
| Convert `Result` to `Option` | `ok`, `err` |

### Custom error types

For libraries, define an enum implementing `std::error::Error + Display + Debug`. Tools:

- **`thiserror`** — derive macro for nice `Display`/`From` impls.
- **`anyhow`** — type-erased dynamic errors, ideal for binaries.

```rust
use thiserror::Error;

#[derive(Debug, Error)]
enum AppError {
    #[error("io error: {0}")]
    Io(#[from] std::io::Error),
    #[error("invalid config: {0}")]
    Config(String),
}
```

### When to `unwrap`?

- Tests, prototypes, examples.
- Cases where an invariant truly cannot fail — prefer `expect("explain why")` instead of `unwrap()` so the panic message documents the assumption.

## Quiz

1. What two things does the `?` operator do?
2. Convert this to use `?`:
   ```rust
   fn parse_and_double(s: &str) -> Result<i32, std::num::ParseIntError> {
       match s.parse::<i32>() {
           Ok(n) => Ok(n * 2),
           Err(e) => Err(e),
       }
   }
   ```
3. Why does this fail to compile?
   ```rust
   fn main() {
       let n: i32 = "x".parse()?;
   }
   ```
4. Difference between `unwrap()` and `expect("msg")`?
5. What's wrong with making *every* function return `panic!` instead of `Result`?
6. Sketch a `Result`-returning function `divide(a: i32, b: i32)` where division by zero is an `Err("div by zero")`. Then call it with `?` from another function.

<details>
<summary>Answers</summary>

1. (a) If the value is `Ok(x)`/`Some(x)`, it evaluates to `x`. (b) Otherwise it short-circuits the enclosing function, returning the error variant — applying `From::from` to convert the error type if needed.
2. ```rust
   fn parse_and_double(s: &str) -> Result<i32, std::num::ParseIntError> {
       let n = s.parse::<i32>()?;
       Ok(n * 2)
   }
   ```
3. `?` requires the enclosing function to return `Result` (or `Option`/something `FromResidual`). `main()` returns `()`. Either change `main` to `fn main() -> Result<(), Box<dyn std::error::Error>>` or handle the result explicitly.
4. Both panic on `Err`/`None`. `expect` lets you provide a custom panic message that documents *why* you believed it could not fail.
5. You lose the ability to handle errors. Callers cannot recover, retry, or transform. `panic!` is for true bugs / unrecoverable program states.
6. ```rust
   fn divide(a: i32, b: i32) -> Result<i32, &'static str> {
       if b == 0 { Err("div by zero") } else { Ok(a / b) }
   }

   fn run() -> Result<(), &'static str> {
       let q = divide(10, 0)?;
       println!("{q}");
       Ok(())
   }
   ```
</details>
