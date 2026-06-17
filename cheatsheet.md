# Rust Cheatsheet

A condensed reference. Skim it before quizzes.

## Bindings

```rust
let x = 5;                    // immutable
let mut x = 5;                // mutable
let x = 5;                    // shadowing (allows new type)
const MAX: u32 = 100;         // compile-time constant, SCREAMING_SNAKE
static GREETING: &str = "hi"; // 'static lifetime
```

## Scalar types

`i8..i128`, `u8..u128`, `isize`, `usize`, `f32`, `f64`, `bool`, `char`.
Default int: `i32`. Default float: `f64`.

## Strings

```rust
let lit: &str = "hello";              // string slice (often 'static)
let s: String = String::from("hi");   // owned
let n = s.len();                      // bytes, not chars
for c in s.chars() {}                 // iterate Unicode scalar values
let n: i32 = "42".parse().unwrap();   // &str -> i32
let s = format!("x={x}");             // build owned String
```

## Collections

```rust
let v: Vec<i32> = vec![1, 2, 3];
v.iter() / v.iter_mut() / v.into_iter()

use std::collections::HashMap;
let mut m = HashMap::new();
m.insert("a", 1);
*m.entry("a").or_insert(0) += 1;
```

## Control flow

```rust
let x = if cond { 1 } else { 2 };
let x = loop { break 42; };

for i in 0..10 {}                     // exclusive
for i in 0..=10 {}                    // inclusive
'outer: loop { break 'outer; }
```

## Pattern matching

```rust
match x {
    0 => "zero",
    1 | 2 => "one or two",
    3..=9 => "small",
    n if n < 0 => "negative",
    _ => "other",
}

if let Some(v) = opt {}
while let Some(v) = stack.pop() {}
let Some(v) = opt else { return; };
```

## Functions & closures

```rust
fn add(a: i32, b: i32) -> i32 { a + b }
let f = |a, b| a + b;
let g = move |x| println!("{captured} {x}");
```

## Structs & enums

```rust
struct User { name: String, active: bool }
struct Point(f64, f64);
struct Unit;

impl User {
    fn new(name: String) -> Self { Self { name, active: true } }
    fn name(&self) -> &str { &self.name }
    fn deactivate(&mut self) { self.active = false; }
    fn into_name(self) -> String { self.name }
}

enum Shape {
    Circle(f64),
    Rect { w: f64, h: f64 },
    Unit,
}
```

## Generics & traits

```rust
fn largest<T: PartialOrd>(xs: &[T]) -> &T { /* ... */ }

trait Area {
    fn area(&self) -> f64;
    fn name(&self) -> &str { "shape" }   // default
}

impl Area for Circle { fn area(&self) -> f64 { /* ... */ } }

fn print_area(a: &impl Area) {}          // static dispatch
fn print_area_dyn(a: &dyn Area) {}       // dynamic dispatch
```

## Lifetimes

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { /* ... */ }
struct Excerpt<'a> { part: &'a str }
```

Elision: one input ref → output borrows from it; `&self` → output borrows from `self`.

## Error handling

```rust
fn read() -> Result<String, io::Error> {
    let s = fs::read_to_string("x.txt")?;
    Ok(s)
}

opt.unwrap_or(default)
opt.unwrap_or_else(|| compute())
res.map_err(|e| MyErr::from(e))
opt.ok_or("missing")
```

## Iterators (common adaptors)

```rust
v.iter()
 .filter(|&&x| x > 0)
 .map(|&x| x * 2)
 .take(10)
 .sum::<i32>();

(1..=10).collect::<Vec<_>>();
v.iter().enumerate().for_each(|(i, x)| println!("{i}: {x}"));
```

## Smart pointers

| Use case | Type |
|---|---|
| Heap-allocate / recursive | `Box<T>` |
| Shared ownership (1 thread) | `Rc<T>` |
| Shared ownership (threads) | `Arc<T>` |
| Mutate via shared ref | `RefCell<T>` / `Cell<T>` |
| Sync mutation (threads) | `Mutex<T>` / `RwLock<T>` |

## Concurrency

```rust
use std::thread;
let h = thread::spawn(move || { /* work */ });
h.join().unwrap();

use std::sync::{Arc, Mutex, mpsc};
let (tx, rx) = mpsc::channel();
```

## Async (with tokio)

```rust
#[tokio::main]
async fn main() {
    let body = fetch().await;
    let (a, b) = tokio::join!(fetch_a(), fetch_b());
    let task = tokio::spawn(async { /* ... */ });
    task.await.unwrap();
}
```

## Cargo commands

```
cargo new app           cargo new --lib mylib
cargo run               cargo build [--release]
cargo check             cargo test [name] [-- --nocapture]
cargo doc --open        cargo fmt           cargo clippy
cargo add serde --features derive
cargo update            cargo tree
```

## Common attributes

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash, Default)]
#[cfg(test)] mod tests { /* ... */ }
#[allow(dead_code)] #[deny(warnings)]
#[inline] #[must_use] #[non_exhaustive]
```
