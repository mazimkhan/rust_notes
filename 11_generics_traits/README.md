# 11 — Generics & Traits

## Concept summary

### Generics

Parametrize types and functions over a *type variable*.

```rust
fn largest<T: PartialOrd>(items: &[T]) -> &T {
    let mut best = &items[0];
    for x in items {
        if x > best { best = x; }
    }
    best
}

struct Pair<T, U> { first: T, second: U }
```

- **Monomorphization**: the compiler generates a specialized copy of the generic code for each concrete type used → zero runtime cost.

### Traits

A *trait* is a set of methods a type can implement (similar to an interface / type class).

```rust
trait Greet {
    fn hello(&self) -> String;                 // required
    fn shout(&self) -> String {                // default method
        self.hello().to_uppercase()
    }
}

struct En;
impl Greet for En {
    fn hello(&self) -> String { "hi".into() }
}
```

### Trait bounds

```rust
fn print_all<T: std::fmt::Display>(xs: &[T]) { /* ... */ }

// where-clause when the signature gets noisy
fn process<T, U>(t: T, u: U) -> i32
where
    T: Clone + std::fmt::Debug,
    U: IntoIterator<Item = i32>,
{ /* ... */ }
```

### `impl Trait` vs `dyn Trait`

- `impl Trait` (parameter or return position) — *static dispatch*, monomorphized. Caller sees one concrete (anonymous) type.
- `dyn Trait` — *dynamic dispatch* via a fat pointer (`&dyn Trait`, `Box<dyn Trait>`). Heterogeneous collections, dynamic plug-in points.

```rust
fn make_greeter() -> impl Greet { En }         // static
fn registry() -> Vec<Box<dyn Greet>> { vec![] } // dynamic
```

### Common standard traits

`Debug`, `Display`, `Clone`, `Copy`, `Default`, `PartialEq`/`Eq`, `Hash`, `Ord`/`PartialOrd`, `From`/`Into`, `Iterator`, `Drop`.

## Quiz

1. What does *monomorphization* mean for generics in Rust?
2. Why does `largest` above need `T: PartialOrd`?
3. Difference between `impl Trait` and `dyn Trait` in return position?
4. Will this compile? Fix it.
   ```rust
   fn add<T>(a: T, b: T) -> T { a + b }
   ```
5. Implement a trait `Area` with a method `area(&self) -> f64`. Provide impls for `Circle { r: f64 }` and `Square { side: f64 }`.
6. Why can't a function return `Vec<impl Trait>` and put two different concrete types in it? What do you use instead?

<details>
<summary>Answers</summary>

1. For each concrete type the generic is instantiated with, the compiler emits a specialized copy of the code at compile time. Generics have **zero runtime cost** but can grow binary size.
2. Comparing values with `>` requires the `PartialOrd` trait to be in scope for `T`. Without the bound, the body wouldn't type-check.
3. `impl Trait`: one specific (compiler-chosen) type behind an opaque alias — static dispatch, no allocation. `dyn Trait`: type-erased trait object via vtable — supports heterogeneous values, needs indirection (`&dyn`, `Box<dyn>`).
4. Needs a bound that supports `+`. `fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T { a + b }`.
5. ```rust
   trait Area { fn area(&self) -> f64; }

   struct Circle { r: f64 }
   struct Square { side: f64 }

   impl Area for Circle { fn area(&self) -> f64 { std::f64::consts::PI * self.r * self.r } }
   impl Area for Square { fn area(&self) -> f64 { self.side * self.side } }
   ```
6. `impl Trait` in return position is **one** opaque concrete type. A `Vec` requires a single element type. Use `Vec<Box<dyn Trait>>` for heterogeneous collections.
</details>
