# 19 — Macros & Unsafe Rust

## Concept summary

### Declarative macros (`macro_rules!`)

Pattern-based code expansion at compile time.

```rust
macro_rules! square {
    ($x:expr) => { $x * $x };
}

let r = square!(3 + 1);   // expands to (3 + 1) * (3 + 1) = 16
```

- Common *fragment specifiers*: `expr`, `ident`, `ty`, `path`, `pat`, `stmt`, `block`, `tt`, `literal`, `meta`.
- Repetition: `$( ... ),*` or `$( ... );+`.

```rust
macro_rules! vec_of {
    ( $( $x:expr ),* $(,)? ) => {{
        let mut v = Vec::new();
        $( v.push($x); )*
        v
    }};
}
let v: Vec<i32> = vec_of![1, 2, 3];
```

### Procedural macros

Generated from a token stream → token stream by a separate **proc-macro crate**. Three kinds:

1. `#[derive(MyTrait)]` — like `#[derive(Debug)]`.
2. **Attribute** macros — `#[my_attr] fn foo() { ... }`.
3. **Function-like** macros — `sql!(...)`, `html!(...)`.

Common helpers: [`syn`](https://crates.io/crates/syn), [`quote`](https://crates.io/crates/quote), [`proc-macro2`](https://crates.io/crates/proc-macro2). Most people derive via [`thiserror`](https://crates.io/crates/thiserror), [`serde`](https://crates.io/crates/serde), [`clap`](https://crates.io/crates/clap) before writing one themselves.

### Unsafe Rust

`unsafe` opts in to **five superpowers** the compiler can't verify:

1. Dereference raw pointers (`*const T`, `*mut T`).
2. Call `unsafe` functions (including FFI).
3. Read/write `static mut` variables.
4. Implement `unsafe` traits.
5. Access fields of `union`s.

```rust
let mut x = 5;
let r1 = &x as *const i32;
let r2 = &mut x as *mut i32;
unsafe {
    println!("{}", *r1);
    *r2 = 10;
}
```

`unsafe` does **not** disable the borrow checker or type checker. It's a contract that **you, the programmer**, uphold the invariants the compiler can no longer verify. Wrap unsafe blocks in safe APIs that maintain those invariants externally.

### FFI (C interop)

```rust
extern "C" {
    fn abs(input: i32) -> i32;        // declare a C function
}

#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 { a + b }   // expose to C

unsafe { println!("{}", abs(-3)); }
```

Add a `build.rs` and `cc`/`bindgen` for serious bindings.

### When NOT to reach for `unsafe`

Almost always there's a safe alternative. Use `unsafe` only when you've measured a real need (performance, FFI, OS primitives) and you can articulate the invariants you're upholding.

## Quiz

1. What does `$x:expr` mean in a `macro_rules!` matcher?
2. Why is `vec!` a macro and not a function?
3. List two of the five superpowers `unsafe` enables.
4. Does writing `unsafe { ... }` turn off the borrow checker?
5. What's a safe API wrapping an internal `unsafe` block? Give one stdlib example.
6. Write a tiny `macro_rules!` macro `min!(a, b)` that returns the smaller of two `Ord` values.

<details>
<summary>Answers</summary>

1. It captures one *expression* (a Rust syntactic category) and binds it to `$x` for use in the expansion.
2. `vec![1, 2, 3]` takes a variable number of arguments of (potentially) different syntactic shapes and expands to a block of code. Regular Rust functions can't take variadic arguments.
3. Any two of: deref raw pointers, call `unsafe` functions, access `static mut`, implement an `unsafe` trait, access `union` fields.
4. **No.** It only enables the five superpowers above. Type checking, borrow checking, and lifetime rules still apply.
5. `Vec::push` internally uses `unsafe` to write to uninitialized capacity, but its safe signature upholds the invariant (it only writes within allocated, owned memory). `Mutex`, `RefCell`, `String::from_utf8_unchecked`'s safe counterparts, etc., follow the same pattern.
6. ```rust
   macro_rules! min {
       ($a:expr, $b:expr) => {{
           let a = $a; let b = $b;
           if a < b { a } else { b }
       }};
   }
   ```
</details>
