# 14 — Testing & Documentation

## Concept summary

### Test kinds

- **Unit tests** — live alongside code in `#[cfg(test)] mod tests { ... }`. Can access private items.
- **Integration tests** — live in `tests/` at the crate root. One file = one separate binary; can only call the **public** API.
- **Doc tests** — code in `///` doc comments is compiled and executed by `cargo test`.

### Unit test layout

```rust
pub fn add(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn adds_positive() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    #[should_panic(expected = "div by zero")]
    fn panics_on_zero() { /* ... */ }

    #[test]
    fn returns_result() -> Result<(), String> {
        if add(2, 2) == 4 { Ok(()) } else { Err("nope".into()) }
    }
}
```

### Useful macros

`assert!`, `assert_eq!`, `assert_ne!`, plus the `#[should_panic]` attribute. Custom messages: `assert_eq!(a, b, "context: {a:?} vs {b:?}");`.

### Running tests

```bash
cargo test                    # run all
cargo test add                # filter by name
cargo test -- --nocapture     # show println! output
cargo test --release          # optimized
```

Tests run in **parallel** by default — keep them independent.

### Documentation

Use `///` (item docs) and `//!` (module/crate docs). Markdown is supported.

```rust
/// Adds two numbers.
///
/// # Examples
///
/// ```
/// use mylib::add;
/// assert_eq!(add(2, 3), 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

Generate & open: `cargo doc --open`.

### Doc test attributes

- ` ```no_run ` — compile but don't run.
- ` ```ignore ` — don't compile.
- ` ```should_panic ` — must panic to pass.

## Quiz

1. Where do you put unit tests vs. integration tests, and what's the visibility difference?
2. What attribute marks a function as a test?
3. How would you write a test for a function that should panic with the message `"empty"`?
4. What does `cargo test -- --nocapture` do?
5. Why are doc-tests useful beyond ordinary tests?
6. Write a doc comment with an example for this function:
   ```rust
   pub fn double(n: i32) -> i32 { n * 2 }
   ```

<details>
<summary>Answers</summary>

1. **Unit tests** live in the same file as the code, inside `#[cfg(test)] mod tests { ... }`, and can access private items. **Integration tests** live in `tests/`, each file compiles as a separate crate that only sees the public API.
2. `#[test]` above an `fn` (with no parameters, returning `()` or `Result<(), E>` where `E: Debug`).
3. ```rust
   #[test]
   #[should_panic(expected = "empty")]
   fn fails_on_empty() { /* call code that panics with "empty" */ }
   ```
4. By default `cargo test` captures `stdout`/`stderr`. The `--nocapture` flag (passed through to the test runner) lets prints reach the terminal so you can debug.
5. They double as **executable documentation** — the example you wrote in the docs is guaranteed to compile and produce the asserted result, so docs can never drift from code.
6. ```rust
   /// Doubles `n`.
   ///
   /// # Examples
   ///
   /// ```
   /// assert_eq!(my_crate::double(3), 6);
   /// ```
   pub fn double(n: i32) -> i32 { n * 2 }
   ```
</details>
