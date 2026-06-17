# Rust Learning Plan

A 20-module curriculum from beginner to advanced. Each module has a short
concept summary and a quiz (with collapsible answers) so you can practice
immediately. Treat the modules as a checklist — do them in order.

## How to use this repo

1. Install Rust via [rustup](https://rustup.rs): `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
2. Verify: `rustc --version && cargo --version`
3. Open each module folder in order and read its `README.md`.
4. Try the quiz **before** revealing the answer.
5. For code-style questions, spin up a scratch project: `cargo new scratch && cd scratch` and paste snippets into `src/main.rs`.
6. Tick the box in the checklist below as you finish each module.

Tip: keep the [Rust Book](https://doc.rust-lang.org/book/) and
[Rust by Example](https://doc.rust-lang.org/rust-by-example/) open as
companion references.

---

## Phase 1 — Fundamentals

- [ ] **01.** [Hello, Cargo!](01_hello_cargo/README.md) — toolchain, `cargo new`, `cargo run`, `cargo check`
- [ ] **02.** [Variables, Mutability & Constants](02_variables_mutability/README.md) — `let`, `mut`, shadowing, `const`, `static`
- [ ] **03.** [Data Types](03_data_types/README.md) — scalars, tuples, arrays, type inference & annotation
- [ ] **04.** [Functions & Control Flow](04_functions_control_flow/README.md) — `fn`, expressions vs statements, `if`, `loop`, `while`, `for`
- [ ] **05.** [Ownership & Move Semantics](05_ownership/README.md) — three rules, `Copy` vs `Move`, `Drop`
- [ ] **06.** [References, Borrowing & Slices](06_references_borrowing/README.md) — `&T`, `&mut T`, aliasing rules, string & array slices

## Phase 2 — Intermediate

- [ ] **07.** [Structs & Methods](07_structs/README.md) — named/tuple/unit structs, `impl`, associated functions
- [ ] **08.** [Enums & Pattern Matching](08_enums_pattern_matching/README.md) — `Option`, `match`, `if let`, `while let`
- [ ] **09.** [Common Collections](09_collections/README.md) — `Vec<T>`, `String`, `HashMap<K,V>`
- [ ] **10.** [Error Handling](10_error_handling/README.md) — `panic!`, `Result`, `?` operator, error conversion
- [ ] **11.** [Generics & Traits](11_generics_traits/README.md) — generic functions/types, trait definitions, bounds, default methods
- [ ] **12.** [Lifetimes](12_lifetimes/README.md) — annotation syntax, the three elision rules, `'static`

## Phase 3 — Advanced

- [ ] **13.** [Modules, Crates & Workspaces](13_modules_crates/README.md) — `mod`, `pub`, `use`, paths, Cargo workspaces
- [ ] **14.** [Testing & Documentation](14_testing_docs/README.md) — `#[test]`, integration tests, doc tests, `rustdoc`
- [ ] **15.** [Closures & Iterators](15_closures_iterators/README.md) — `Fn`/`FnMut`/`FnOnce`, lazy iterators, `map`/`filter`/`fold`
- [ ] **16.** [Smart Pointers & Interior Mutability](16_smart_pointers/README.md) — `Box`, `Rc`, `Arc`, `RefCell`, `Cell`, `Deref`, `Drop`
- [ ] **17.** [Fearless Concurrency](17_concurrency/README.md) — threads, channels, `Mutex`, `Send`/`Sync`
- [ ] **18.** [Async / Await](18_async_await/README.md) — futures, executors, `tokio` basics, `.await`

## Phase 4 — Mastery & Projects

- [ ] **19.** [Macros & Unsafe Rust](19_macros_unsafe/README.md) — `macro_rules!`, proc macros, raw pointers, FFI, `unsafe` invariants
- [ ] **20.** [Capstone Projects](20_capstone/README.md) — CLI tool, mini web service, mini interpreter

---

## Suggested pace

- **Casual (12 weeks):** 1–2 modules / week.
- **Intensive (4 weeks):** 5 modules / week.
- **Deep dive (6 months):** add one open-source contribution and one toy project per phase.

## Project ideas to reinforce concepts

| Phase | Project |
| --- | --- |
| After Phase 1 | Temperature converter CLI, FizzBuzz, guessing game |
| After Phase 2 | Note-taking CLI with `Vec<Struct>`, word-count tool |
| After Phase 3 | Multi-threaded log parser, mini key-value store, simple HTTP client with `reqwest` |
| Phase 4 capstone | Async web scraper, terminal Pomodoro, JSON parser, mini shell |
