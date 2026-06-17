# 02 — Variables, Mutability & Constants

## Concept summary

- Bindings are **immutable by default**: `let x = 5;` — reassigning is a compile error.
- Use `mut` to make a binding mutable: `let mut x = 5; x = 6;`.
- **Shadowing** lets you re-declare a name with `let` again — possibly with a different type. The old binding is shadowed, not mutated.
  ```rust
  let n = "42";            // &str
  let n: i32 = n.parse().unwrap(); // i32, shadows previous n
  ```
- `const`:
  - Always immutable, no `mut` allowed.
  - Type annotation **required**: `const MAX: u32 = 100_000;`.
  - Must be a *const expression* (evaluated at compile time).
  - Convention: `SCREAMING_SNAKE_CASE`.
- `static`:
  - Has a fixed memory address for the whole program (`'static` lifetime).
  - `static mut` exists but is `unsafe`. Prefer interior mutability or atomics instead.

| | `let` | `let mut` | `const` | `static` |
|---|---|---|---|---|
| Mutable? | no | yes | never | only via `unsafe`/atomics |
| Type annotation required? | optional | optional | **required** | **required** |
| Evaluated when? | runtime | runtime | compile time | compile time |

## Quiz

1. Will this compile? Why or why not?
   ```rust
   let x = 5;
   x = 6;
   ```
2. What is the difference between **shadowing** and `mut`?
3. Predict the output:
   ```rust
   let spaces = "   ";
   let spaces = spaces.len();
   println!("{spaces}");
   ```
4. Which of these declarations is invalid?
   - `const MAX = 100;`
   - `const MAX: u32 = 100;`
   - `let MAX: u32 = 100;`
5. Why is `static mut` discouraged?
6. Write a snippet that converts a `&str` containing a number to an `i32` using shadowing.

<details>
<summary>Answers</summary>

1. **No.** `x` is immutable; reassignment is a compile error. Either declare with `let mut x = 5;` or shadow with `let x = 6;`.
2. Shadowing creates a **new binding** (possibly a different type) and the old value can still be dropped/used until shadowed. `mut` reassigns the **same** binding — the type must stay the same.
3. `3` — the second `let` shadows `spaces` with the integer length.
4. `const MAX = 100;` — `const` requires an explicit type annotation.
5. Mutable global state can be accessed concurrently from multiple threads, breaking Rust's data-race guarantees. The compiler requires `unsafe` to acknowledge that. Use `Mutex`, `RwLock`, or `AtomicXxx` instead.
6. ```rust
   let n = "42";
   let n: i32 = n.parse().expect("not a number");
   ```
</details>
