# 09 — Common Collections

`Vec<T>`, `String`, and `HashMap<K, V>` cover ~90% of day-to-day Rust.

## `Vec<T>` — growable array on the heap

```rust
let mut v: Vec<i32> = Vec::new();
v.push(1); v.push(2);

let v = vec![10, 20, 30];           // macro literal
let first: &i32 = &v[0];            // panics if OOB
let maybe: Option<&i32> = v.get(0); // safe
for x in &v { println!("{x}"); }    // iterate by ref
```

- `push` may reallocate, invalidating existing references. The borrow checker enforces that you cannot push while you hold a reference into the vec.

## `String` & `&str`

- `String` is `Vec<u8>` guaranteed to be valid UTF-8.
- Append: `s.push_str("more")`, `s.push('!')`, `s + &other`, `format!("...{x}")`.
- Indexing by `s[0]` is **not allowed** — bytes ≠ characters in UTF-8. Iterate with `.chars()` or `.bytes()`.

```rust
let s = String::from("héllo");
for c in s.chars() { println!("{c}"); }    // 5 chars
println!("{}", s.len());                   // 6 (bytes!)
```

## `HashMap<K, V>`

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert("Ada", 90);
scores.insert("Grace", 95);

let v = scores.get("Ada");                 // Option<&i32>
*scores.entry("Ada").or_insert(0) += 1;    // upsert pattern
for (k, v) in &scores { /* ... */ }
```

- Keys must implement `Eq + Hash`.
- Iteration order is **not** stable across runs.
- For ordered keys use `BTreeMap`.

## Quiz

1. Why is `s[0]` for a `String` a compile error?
2. What's the difference between `v[0]` and `v.get(0)`?
3. What does this print?
   ```rust
   let s = String::from("日本語");
   println!("{} {}", s.len(), s.chars().count());
   ```
4. Why does this fail?
   ```rust
   let mut v = vec![1, 2, 3];
   let first = &v[0];
   v.push(4);
   println!("{first}");
   ```
5. Write idiomatic code that counts how many times each word appears in `let text = "a b a c b a";` using a `HashMap<&str, i32>`.
6. When would you prefer `BTreeMap` over `HashMap`?

<details>
<summary>Answers</summary>

1. UTF-8 characters can be multiple bytes, so `s[0]` is ambiguous (byte? char? grapheme?). Rust forbids it. Use `s.chars()`, `s.bytes()`, or byte slicing on known boundaries.
2. `v[0]` panics if out of bounds and returns `T` (or `&T` via auto-deref); `v.get(0)` is safe and returns `Option<&T>`.
3. `9 3` — three CJK chars at 3 bytes each.
4. `push` may reallocate the backing buffer, which would invalidate `first`. The borrow checker rejects mutation while a shared borrow is alive.
5. ```rust
   use std::collections::HashMap;
   let text = "a b a c b a";
   let mut counts: HashMap<&str, i32> = HashMap::new();
   for word in text.split_whitespace() {
       *counts.entry(word).or_insert(0) += 1;
   }
   ```
6. When you need keys iterated in **sorted order**, or want range queries (`range(..)`). `HashMap` is unordered.
</details>
