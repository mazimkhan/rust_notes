# 07 — Structs & Methods

## Concept summary

Three flavors:

```rust
struct User { name: String, active: bool }     // named-field
struct Point(f64, f64);                        // tuple struct
struct Marker;                                 // unit struct (zero-sized)
```

### Construction & update syntax

```rust
let u1 = User { name: "Ada".into(), active: true };
let u2 = User { name: "Grace".into(), ..u1 };   // copy remaining fields from u1
```

The `..u1` shorthand **moves** non-`Copy` fields from `u1`, so `u1.name` is no longer usable afterward (other `Copy` fields still are).

### Methods (`impl` blocks)

```rust
impl Point {
    // Associated function (no `self`) — typically a constructor.
    fn new(x: f64, y: f64) -> Self { Self(x, y) }

    // Method: takes `&self` to read.
    fn norm(&self) -> f64 { (self.0 * self.0 + self.1 * self.1).sqrt() }

    // Method that mutates.
    fn scale(&mut self, k: f64) { self.0 *= k; self.1 *= k; }

    // Consuming method.
    fn into_tuple(self) -> (f64, f64) { (self.0, self.1) }
}
```

Call associated functions with `Type::name`: `Point::new(1.0, 2.0)`.

### Useful derives

```rust
#[derive(Debug, Clone, PartialEq)]
struct User { /* ... */ }
```

- `Debug` → `{:?}` printing.
- `Clone` / `Copy` → explicit / implicit duplication.
- `PartialEq` / `Eq` → `==`.
- `Hash` → use as `HashMap` key.

## Quiz

1. What is a *unit struct* and when is it useful?
2. After this code, can you still use `u1.name`?
   ```rust
   let u1 = User { name: String::from("Ada"), active: true };
   let u2 = User { active: false, ..u1 };
   ```
3. Difference between `fn new(...) -> Self` and `fn area(&self) -> f64`?
4. Add the minimum derive so `println!("{:?}", p);` works for `Point`.
5. Fix this:
   ```rust
   impl Point {
       fn shift(self, dx: f64) {
           self.0 += dx;
       }
   }
   ```
6. Write a `Rectangle { width, height }` struct with a method `area(&self) -> u32` and an associated function `square(side: u32) -> Self`.

<details>
<summary>Answers</summary>

1. A struct with no fields, e.g. `struct Marker;`. Useful as a *type-level* marker — for trait implementations, generic phantom types, or signaling without storing data.
2. **No.** `u1.name` was moved into `u2` via the `..u1` update. `u1.active` (a `Copy` field) is still usable — but most often the whole `u1` is considered partially moved.
3. `new` is an **associated function** (no `self` receiver); call as `Point::new(...)`. `area` is a **method** that borrows the instance; call as `p.area()`.
4. `#[derive(Debug)]` above the struct.
5. `self` is taken by value (consumed) **and** the body tries to mutate it. Change to `&mut self`:
   ```rust
   fn shift(&mut self, dx: f64) { self.0 += dx; }
   ```
6. ```rust
   struct Rectangle { width: u32, height: u32 }

   impl Rectangle {
       fn area(&self) -> u32 { self.width * self.height }
       fn square(side: u32) -> Self { Self { width: side, height: side } }
   }
   ```
</details>
