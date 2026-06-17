# 16 — Smart Pointers & Interior Mutability

## Concept summary

Smart pointers are structs that *act like pointers* but add behavior via the `Deref` and `Drop` traits.

| Type | Ownership | Threading | Use when… |
|---|---|---|---|
| `Box<T>` | single owner, on heap | single-threaded | move large data to heap; recursive types; trait objects |
| `Rc<T>` | shared, reference-counted | **single-threaded** | shared immutable graphs/trees |
| `Arc<T>` | shared, atomic refcount | thread-safe | shared data across threads |
| `RefCell<T>` | single owner, runtime-checked borrowing | single-threaded | "interior mutability" through an immutable handle |
| `Cell<T>` | single owner, `Copy` only | single-threaded | small `Copy` values mutated through `&` |
| `Mutex<T>` / `RwLock<T>` | shared mutable | thread-safe | shared mutable state across threads (module 17) |

### `Box<T>`

```rust
let b = Box::new(42);
println!("{}", *b);        // Deref to &i32

// Recursive type — sized via Box
enum List { Cons(i32, Box<List>), Nil }
```

### `Rc<T>` for shared ownership

```rust
use std::rc::Rc;
let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a);     // cheap: bumps refcount, does NOT deep-copy
let c = a.clone();         // same; both are idiomatic
println!("{}", Rc::strong_count(&a));   // 3
```

### `RefCell<T>` — interior mutability

`RefCell` enforces the borrow rules **at runtime** rather than compile time. Borrowing wrongly → panics.

```rust
use std::cell::RefCell;
let v = RefCell::new(vec![1, 2, 3]);
v.borrow_mut().push(4);                // &mut access through &
println!("{:?}", v.borrow());          // &access
```

Common combo: `Rc<RefCell<T>>` — multiple owners that can each mutate the inner value (single-threaded). Across threads: `Arc<Mutex<T>>`.

### `Deref` & `Drop`

- `Deref::deref(&self) -> &Self::Target` enables `*box` and **deref coercion** (`&String` → `&str`, `&Vec<T>` → `&[T]`, `&Box<T>` → `&T`).
- `Drop::drop(&mut self)` runs when the value goes out of scope. You can't call `drop` yourself directly — use `std::mem::drop(x)` to drop early.

### Reference cycles

`Rc` can leak via cycles. Break them with `Weak<T>` (non-owning).

```rust
use std::rc::{Rc, Weak};
struct Node { parent: Weak<Node>, children: Vec<Rc<Node>> }
```

## Quiz

1. When would you use `Box<T>` instead of putting a value on the stack directly?
2. Why is `Rc<T>` not safe across threads? What do you use instead?
3. What does `RefCell` give you that ordinary borrowing rules don't, and at what cost?
4. Predict — does this compile? What happens at runtime?
   ```rust
   use std::cell::RefCell;
   let r = RefCell::new(5);
   let a = r.borrow();
   let b = r.borrow_mut();
   println!("{} {}", a, b);
   ```
5. Why do reference cycles with `Rc<T>` leak, and what's the standard fix?
6. Sketch a `Rc<RefCell<i32>>` shared between two variables `a` and `b`, where mutating through `a` is visible through `b`.

<details>
<summary>Answers</summary>

1. (a) The value is huge and you want to avoid copying it. (b) The type is recursive (its size isn't known statically) — `Box` provides indirection of known size. (c) You need a trait object: `Box<dyn Trait>`.
2. `Rc` increments/decrements its refcount **non-atomically**, so concurrent updates would race. Use `Arc<T>` (atomic refcounts) for multi-threaded shared ownership.
3. It lets you mutate data through a shared (`&`) reference — the borrow rules are enforced **dynamically** instead of statically. Cost: borrow conflicts now **panic at runtime** instead of compile-time errors, plus a small bookkeeping overhead.
4. **Compiles.** At runtime the second borrow (`borrow_mut`) panics: "already borrowed" — you cannot hold a shared borrow and an exclusive borrow simultaneously.
5. Each `Rc` participates in a counted graph; a cycle keeps everyone's count ≥ 1 forever, so nothing gets dropped. Fix: use `Weak<T>` for back-pointers (non-owning), e.g. `parent` in a tree.
6. ```rust
   use std::{cell::RefCell, rc::Rc};
   let a = Rc::new(RefCell::new(1));
   let b = Rc::clone(&a);
   *a.borrow_mut() += 10;
   println!("{}", b.borrow());   // 11
   ```
</details>
