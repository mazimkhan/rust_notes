# 20 — Capstone Projects

By now you've covered the language end-to-end. The best way to *retain* it is to build a few small projects. Pick **two or three** below — one CLI, one with concurrency, and one with async.

## Project 1: `grepr` — a tiny grep clone (CLI, I/O, errors)

Concepts: modules 02, 03, 04, 06, 09, 10, 13, 14.

Goals:

- Use [`clap`](https://crates.io/crates/clap) to parse `grepr [-i] [-n] <pattern> <file>`.
- Read the file line-by-line (`std::io::BufRead`).
- Print matching lines; with `-n` prefix the line number; with `-i` do case-insensitive matching.
- Return proper exit codes (`0` matched, `1` no match, `2` error).
- Add integration tests in `tests/` using a fixture file.

Stretch:

- Use [`regex`](https://crates.io/crates/regex) for the pattern.
- Add `-r` recursive directory walking with [`walkdir`](https://crates.io/crates/walkdir).
- Stream from `stdin` when no file is given.

## Project 2: `kv-store` — multi-threaded in-memory key/value store

Concepts: modules 07, 08, 09, 11, 12, 15, 16, 17.

Goals:

- Define `enum Command { Get(String), Set(String, String), Del(String) }`.
- A worker thread owns the `HashMap<String, String>`. Other threads send commands via `mpsc::channel`.
- A `Server` struct exposes safe `set` / `get` / `del` methods backed by the channel.
- Add benchmarks comparing channel-based vs `Arc<Mutex<HashMap>>` designs.

Stretch:

- Persist to disk on shutdown (`serde_json`).
- Add a TTL per entry.

## Project 3: `tinyurl` — async URL shortener (async/await, web)

Concepts: modules 10, 11, 13, 16, 17, 18.

Goals:

- HTTP server with [`axum`](https://crates.io/crates/axum) (or `actix-web` / `warp`).
- Endpoints:
  - `POST /shorten { url }` → returns short code.
  - `GET  /:code` → 302 redirect to the original URL.
- Storage: `Arc<RwLock<HashMap<String, String>>>` (in-memory). Swap to `sled` or `sqlite` for persistence.
- Logging via [`tracing`](https://crates.io/crates/tracing).

Stretch:

- Rate limit per IP.
- Async tests with `#[tokio::test]`.

## Project 4: `calc` — expression interpreter (parser, enums, recursion)

Concepts: modules 05, 07, 08, 10, 11, 15, 16.

Goals:

- Tokenize and parse `1 + 2 * (3 - 4)` into an AST: `enum Expr { Num(f64), Bin(Op, Box<Expr>, Box<Expr>) }`.
- Recursive-descent parser with proper precedence.
- Evaluator returning `Result<f64, EvalError>`.
- Doc tests demonstrating use.

Stretch:

- Variables and assignment.
- Functions and let-bindings.
- A REPL with [`rustyline`](https://crates.io/crates/rustyline).

---

## Checklist when you finish a project

- [ ] All `cargo check`, `cargo clippy`, `cargo fmt --check` pass cleanly.
- [ ] At least one unit test per non-trivial function.
- [ ] At least one integration test.
- [ ] `cargo doc --open` renders meaningful docs for every `pub` item.
- [ ] README explains how to build, run, and test.

## What to read next

- [Rust Atomics and Locks](https://marabos.nl/atomics/) — Mara Bos (free online).
- [Rust for Rustaceans](https://nostarch.com/rust-rustaceans) — Jon Gjengset.
- [Async book](https://rust-lang.github.io/async-book/) and [Tokio tutorial](https://tokio.rs/tokio/tutorial).
- *The Rustonomicon* — for unsafe deep dives.
- Pick a real OSS Rust project and read its source: `ripgrep`, `tokio`, `axum`, `bat`, `fd`.
