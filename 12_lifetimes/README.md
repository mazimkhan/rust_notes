# 12 — Lifetimes

## Concept summary

A **lifetime** is a compile-time label describing *how long a reference is valid*. The borrow checker uses lifetimes to prove references never outlive their data.

You write `'a` (any lowercase identifier) as a generic over lifetimes:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() >= y.len() { x } else { y }
}
```

> "For any lifetime `'a`, given two refs valid for `'a`, the returned ref is also valid for `'a`."

### Three elision rules (when you can *omit* lifetimes)

The compiler infers lifetimes automatically when:

1. **Each input** reference gets its own lifetime parameter.
2. If there's **exactly one input lifetime**, it's assigned to all outputs.
3. If one of the inputs is `&self` or `&mut self`, **the lifetime of `self`** is assigned to all outputs.

Examples where elision works:

```rust
fn first_word(s: &str) -> &str { /* ... */ }     // rule 2
impl Foo { fn name(&self) -> &str { /* ... */ } } // rule 3
```

Where it does *not* work — you must annotate:

```rust
fn longest(x: &str, y: &str) -> &str { ... }     // ambiguous output lifetime
// fix:
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { ... }
```

### Lifetimes in structs

If a struct holds a reference, declare its lifetime:

```rust
struct Excerpt<'a> { part: &'a str }
```

The struct cannot outlive the data its fields reference.

### `'static`

The special lifetime meaning "lives for the entire program". String literals (`&'static str`) have it. Don't sprinkle it as a fix — it usually masks a real ownership problem.

### Mental model

Lifetimes don't *cause* anything to live longer. They are **descriptions** of relationships that already exist; the compiler refuses code that would violate them.

## Quiz

1. Why is `fn longest(x: &str, y: &str) -> &str { … }` rejected, but `fn first_word(s: &str) -> &str { … }` accepted?
2. Add lifetimes to compile:
   ```rust
   struct Important { excerpt: &str }
   ```
3. What does `'static` mean? Name a common value with that lifetime.
4. Will this compile? Why or why not?
   ```rust
   fn dangle() -> &String {
       let s = String::from("oops");
       &s
   }
   ```
5. Explain elision rule 3 in your own words and give an example where it applies.
6. Predict — does this compile?
   ```rust
   fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
       if x.len() >= y.len() { x } else { y }
   }
   let s1 = String::from("longer one");
   let r;
   {
       let s2 = String::from("hi");
       r = longest(&s1, &s2);
   }
   println!("{r}");
   ```

<details>
<summary>Answers</summary>

1. `first_word` has one input reference, so elision rule 2 ties the output to it. `longest` has two — the compiler doesn't know which input the output borrows from, so you must spell it out.
2. ```rust
   struct Important<'a> { excerpt: &'a str }
   ```
3. `'static` means the reference is valid for the program's entire run. Examples: string literals (`&'static str`), values leaked via `Box::leak`, and `const`/`static` items.
4. **No.** `s` is dropped at the end of `dangle`; returning a reference to it would dangle. The borrow checker rejects this. Return `String` (owned) instead.
5. Rule 3: when a method has `&self` (or `&mut self`) plus other reference params, the output lifetime defaults to `self`'s lifetime. Example: `impl Foo { fn name(&self) -> &str { &self.name } }` — output borrows from `self` implicitly.
6. **No.** The returned reference must live as long as both `s1` and `s2` — i.e. as long as the shorter one, `s2`. But `r` is used after `s2` is dropped, so the compiler rejects it.
</details>
