# 15 — Closures & Iterators

## Concept summary

### Closures

Anonymous functions that can capture their environment.

```rust
let add = |a, b| a + b;
let add: fn(i32, i32) -> i32 = |a, b| a + b;
let mul = |a: i32, b: i32| -> i32 { a * b };
```

Closures implement one of three traits, automatically chosen by the compiler:

| Trait | Captures by | Can be called |
|---|---|---|
| `Fn` | shared reference (`&T`) | many times |
| `FnMut` | exclusive reference (`&mut T`) | many times, needs `&mut` access |
| `FnOnce` | by value (move) | exactly once |

- `Fn ⊂ FnMut ⊂ FnOnce`. Every `Fn` is also `FnMut` and `FnOnce`.
- Prefix with `move` to force capturing by value: `let f = move || println!("{name}");`.

### Iterators

The `Iterator` trait has one required method, `fn next(&mut self) -> Option<Item>`. Everything else (`map`, `filter`, `fold`, ...) is built on it.

```rust
let v = vec![1, 2, 3, 4, 5];

let sum_of_even_squares: i32 = v.iter()
    .filter(|&&x| x % 2 == 0)
    .map(|&x| x * x)
    .sum();
```

**Lazy:** iterator adaptors do nothing until consumed. Consumers include `collect`, `sum`, `count`, `for`, `fold`, `find`, `any`, `all`, `last`, `min`, `max`, `reduce`.

### Three flavors of iteration

```rust
for x in &v   { /* &T   */ }   // v.iter()
for x in &mut v { /* &mut T */ }  // v.iter_mut()
for x in v    { /* T    */ }   // v.into_iter() — consumes v
```

### Common adaptors

`map`, `filter`, `enumerate`, `zip`, `chain`, `take(n)`, `skip(n)`, `take_while`, `skip_while`, `flat_map`, `flatten`, `windows`, `chunks`, `step_by`, `rev`, `peekable`, `cloned`/`copied`, `inspect`, `dedup`.

`collect` is generic — use turbofish if needed:

```rust
let v: Vec<i32> = (1..=5).collect();
let v = (1..=5).collect::<Vec<_>>();
let m: std::collections::HashMap<i32, i32> = (1..=5).map(|n| (n, n * n)).collect();
```

### Zero-cost?

Yes — iterator chains compile down to tight loops equivalent to (or better than) hand-written ones, thanks to monomorphization and inlining.

## Quiz

1. Difference between `Fn`, `FnMut`, and `FnOnce`?
2. Why does this print `"hi"`?
   ```rust
   let s = String::from("hi");
   let f = move || println!("{s}");
   f();
   ```
3. What does `iter()` give you vs. `into_iter()`?
4. Predict the result:
   ```rust
   let v = vec![1, 2, 3, 4, 5];
   let r: i32 = v.iter().filter(|&&x| x > 2).map(|&x| x * 10).sum();
   ```
5. Why does this not print anything?
   ```rust
   (1..=3).map(|n| println!("{n}"));
   ```
6. Convert this `for` loop to an iterator chain:
   ```rust
   let v = vec![1, 2, 3, 4, 5];
   let mut sum = 0;
   for x in &v { if x % 2 == 1 { sum += x * x; } }
   ```

<details>
<summary>Answers</summary>

1. They differ by how they capture environment: `Fn` borrows shared (`&`), `FnMut` borrows exclusive (`&mut`) and may mutate captured state, `FnOnce` consumes captured values and can be called at most once. The compiler picks the *least restrictive* trait the body needs.
2. The `move` keyword captures `s` *by value*, transferring ownership into the closure. The closure now owns `s`, so it can use it after the original binding scope ends.
3. `iter()` yields `&T` (borrows the collection). `iter_mut()` yields `&mut T`. `into_iter()` yields `T` and **consumes** the collection.
4. `90` — keeps `3, 4, 5`, multiplies → `30, 40, 50`, sum = `120`. *Wait — re-check:* filter keeps elements where `x > 2`: that's `3, 4, 5`. `30 + 40 + 50 = 120`. **Answer: `120`.**
5. Iterators are **lazy**. `map` returns a new iterator but nothing pulls values from it. Use `.for_each(...)` (or replace with `for n in 1..=3 { ... }`).
6. ```rust
   let sum: i32 = v.iter().filter(|x| **x % 2 == 1).map(|x| x * x).sum();
   ```
</details>
