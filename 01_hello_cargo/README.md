# 01 — Hello, Cargo!

## Concept summary

- **rustc** is the compiler; **cargo** is the build tool + package manager. You will almost always use `cargo`.
- A Rust *package* contains one or more *crates*. A binary crate has `src/main.rs`; a library crate has `src/lib.rs`.
- Key commands:
  - `cargo new my_app` — create a binary package (add `--lib` for a library).
  - `cargo run` — compile + run the binary (debug profile).
  - `cargo build` / `cargo build --release` — compile only.
  - `cargo check` — type-check without producing a binary (fast).
  - `cargo fmt` — format source. `cargo clippy` — lint.
- Every program starts at `fn main() { ... }`. `println!` is a **macro** (note the `!`), not a function.
- `Cargo.toml` is the manifest (name, version, dependencies). `Cargo.lock` pins exact dep versions.

Minimal program:

```rust
fn main() {
    println!("Hello, world!");
}
```

## Quiz

1. What is the difference between `cargo build` and `cargo check`?
2. Why does `println!` end with a `!`?
3. Where does Rust look for the entry point of a binary crate?
4. What does `Cargo.lock` do, and should you commit it for a binary crate? What about a library crate?
5. Predict the output:
   ```rust
   fn main() {
       println!("{} + {} = {}", 2, 3, 2 + 3);
   }
   ```
6. Write the `cargo` command that creates a new library crate named `mathy`.

<details>
<summary>Answers</summary>

1. `cargo build` produces a binary; `cargo check` only runs the type-checker/borrow-checker and skips codegen, so it's much faster for iteration.
2. The `!` marks it as a **macro** invocation. Macros expand at compile time and can take a variable number of arguments — regular functions cannot.
3. In `src/main.rs`, at the function `fn main()`.
4. `Cargo.lock` records exact resolved versions of every dependency for reproducible builds. **Commit it for binaries** (apps you ship); **do not commit it for libraries** (so downstream users get the latest compatible versions).
5. `2 + 3 = 5`
6. `cargo new --lib mathy`
</details>
