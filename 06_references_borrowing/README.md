# 06 — References, Borrowing & Slices

## Concept summary

A **reference** lets you use a value *without taking ownership*. Creating one is called **borrowing**.

### Two flavors

- `&T` — *shared* (immutable) reference. You can have **many** simultaneously.
- `&mut T` — *exclusive* (mutable) reference. You can have **exactly one** and **no shared refs** at the same time.

This is the borrow checker's central rule (often called *aliasing XOR mutation*).

```rust
fn len(s: &String) -> usize { s.len() }    // borrow shared
fn push(s: &mut String) { s.push('!'); }   // borrow exclusive

let mut s = String::from("hi");
let n = len(&s);
push(&mut s);
```

### Common pitfalls

- A reference may **not outlive** the value it points to (covered in lifetimes, module 12).
- You cannot create `&mut` while a `&` is still in use.
- Multiple `&mut` simultaneously → compile error.
- Reborrowing happens implicitly: passing `&mut s` to a function gives back the reference when it returns.

### Slices

A slice is a **view** into a contiguous sequence: `&[T]` for arrays/vecs, `&str` for strings.

```rust
let v = vec![1, 2, 3, 4, 5];
let middle: &[i32] = &v[1..4];   // [2, 3, 4]

let s = String::from("hello");
let hello: &str = &s[0..5];      // careful: byte indices, not char indices
```

- `&String` automatically *derefs* to `&str` when called.
- String slicing must land on a UTF-8 char boundary or it panics at runtime.

## Quiz

1. Why does this fail to compile?
   ```rust
   let mut s = String::from("hi");
   let r1 = &s;
   let r2 = &mut s;
   println!("{r1} {r2}");
   ```
2. What about this — does it compile? Why?
   ```rust
   let mut s = String::from("hi");
   let r1 = &s;
   let r2 = &s;
   println!("{r1} {r2}");
   let r3 = &mut s;
   r3.push('!');
   ```
3. Write a function `longest_word(s: &str) -> &str` (signature only — what lifetime does the return value share?).
4. Predict — what does this print?
   ```rust
   fn add_excite(s: &mut String) { s.push('!'); }
   let mut s = String::from("yo");
   add_excite(&mut s);
   add_excite(&mut s);
   println!("{s}");
   ```
5. What's wrong here?
   ```rust
   fn first(v: &Vec<i32>) -> &i32 {
       &v[0]
   }
   let r;
   {
       let v = vec![1, 2, 3];
       r = first(&v);
   }
   println!("{r}");
   ```
6. Take a slice of the middle three elements of `let a = [10, 20, 30, 40, 50];`.

<details>
<summary>Answers</summary>

1. You cannot have an exclusive (`&mut`) reference while a shared (`&`) reference to the same value is still in use. `r1` is used in the `println!` after `r2` is created.
2. **Yes**, it compiles under NLL (non-lexical lifetimes). `r1` and `r2` are last used in the `println!`, then their borrows end. Only after that does `r3 = &mut s` happen — no overlap.
3. `fn longest_word<'a>(s: &'a str) -> &'a str` — the returned slice borrows from `s`, so it shares `s`'s lifetime. (Often lifetime elision lets you write `fn longest_word(s: &str) -> &str`.)
4. `yo!!`
5. `r` outlives `v`. After the inner block ends, `v` is dropped; `r` would be a dangling reference. The borrow checker rejects this at compile time.
6. `let mid = &a[1..4];` — gives `&[20, 30, 40]`.
</details>
