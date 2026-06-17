# Resources

## Free books

- [The Rust Book](https://doc.rust-lang.org/book/) — the official starting point. Read it once cover-to-cover.
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) — runnable snippets for every concept.
- [Rust Atomics and Locks](https://marabos.nl/atomics/) — Mara Bos. Deep on concurrency.
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/) — unsafe Rust + UB hazards.
- [Async Book](https://rust-lang.github.io/async-book/) — futures, executors, pinning.

## Paid books worth it

- *Programming Rust*, 2nd ed. — Jim Blandy, Jason Orendorff, Leonora F. S. Tindall.
- *Rust for Rustaceans* — Jon Gjengset (intermediate→advanced idioms).
- *Zero to Production in Rust* — Luca Palmieri (build a real web service).

## Hands-on exercise tracks

- [Rustlings](https://github.com/rust-lang/rustlings) — small fix-the-code exercises. Do these alongside Phase 1–2.
- [Exercism Rust track](https://exercism.org/tracks/rust) — graded katas with mentors.
- [Codewars Rust kata](https://www.codewars.com/?language=rust).
- [LeetCode Rust](https://leetcode.com/) — for algorithm practice.
- [Advent of Code](https://adventofcode.com/) — annual puzzles; perfect for iterator + parser practice.

## Reference docs

- [`std` API docs](https://doc.rust-lang.org/std/) — always the source of truth.
- [`std::collections` cheat](https://doc.rust-lang.org/std/collections/) — pick the right collection.
- [`docs.rs`](https://docs.rs/) — auto-generated docs for every crate on crates.io.
- [Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/) and the [API Guidelines](https://rust-lang.github.io/api-guidelines/).

## Tooling

- `rustup` — toolchain manager (`rustup update`, `rustup component add clippy`).
- `cargo` subcommands worth installing:
  - `cargo-edit` — `cargo add`/`rm`/`upgrade`.
  - `cargo-watch` — re-run on file change.
  - `cargo-expand` — expand macros to see what they generate.
  - `cargo-nextest` — much faster test runner.
  - `cargo-deny` / `cargo-audit` — supply-chain checks.

## YouTube / talks

- [Jon Gjengset — "Crust of Rust"](https://www.youtube.com/playlist?list=PLqbS7AVVErFiWDOAVrPt7aYmnuuOLYvOa) — live walk-throughs of std types.
- [Ryan Levick](https://www.youtube.com/@RyanLevicksVideos) — beginner-friendly deep dives.
- [Let's Get Rusty](https://www.youtube.com/@letsgetrusty).

## Community

- [Rust Users Forum](https://users.rust-lang.org/) — the friendliest help forum on the internet.
- [r/rust](https://www.reddit.com/r/rust/).
- [This Week in Rust](https://this-week-in-rust.org/) — weekly newsletter.
- [Rust Discord](https://discord.gg/rust-lang).

## Read source

Pick a small high-quality crate and read it. Suggestions:

- [`ripgrep`](https://github.com/BurntSushi/ripgrep) — CLI tool, fantastic code.
- [`bat`](https://github.com/sharkdp/bat) — `cat` clone.
- [`once_cell`](https://github.com/matklad/once_cell) — small, deep code.
- [`tokio`](https://github.com/tokio-rs/tokio) — async runtime internals.
