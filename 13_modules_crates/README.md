# 13 — Modules, Crates & Workspaces

## Concept summary

### The hierarchy

- **Package** — what `cargo new` creates. Has a `Cargo.toml`. Contains one or more crates.
- **Crate** — a compilation unit. Either a *binary* (`src/main.rs` or `src/bin/*.rs`) or a *library* (`src/lib.rs`). Each crate has its own *crate root*.
- **Module** — namespace within a crate. Declared with `mod`.

### Declaring modules

```rust
// src/lib.rs
mod math;            // looks for src/math.rs OR src/math/mod.rs
pub mod utils;       // make it visible to users of the crate
```

```rust
// src/math.rs
pub fn add(a: i32, b: i32) -> i32 { a + b }
mod private;         // src/math/private.rs
```

### Paths & visibility

- Absolute paths: `crate::math::add` (this crate), `std::io::Read` (external crate).
- Relative paths: `self::`, `super::`.
- Items are **private by default**. Use `pub` to expose. Visibility modifiers: `pub`, `pub(crate)`, `pub(super)`, `pub(in path)`.

### `use` for ergonomics

```rust
use std::collections::HashMap;
use crate::math::add;
use crate::math::{self, add as plus};   // grouped + rename
use crate::math::*;                     // glob (prefer for preludes only)
```

### Cargo workspaces

A workspace groups multiple packages that share a `Cargo.lock` and `target/`.

```toml
# Cargo.toml at workspace root
[workspace]
members = ["app", "core", "cli"]
```

Run cross-workspace: `cargo test -p core`, `cargo build --workspace`.

### Dependencies

```toml
# Cargo.toml
[dependencies]
serde = { version = "1", features = ["derive"] }
anyhow = "1"

[dev-dependencies]
proptest = "1"

[features]
default = ["std"]
std = []
```

Use `cargo add <crate>` to add deps; `cargo update` to bump.

## Quiz

1. What's the difference between a *package* and a *crate*?
2. What files does Rust look at when you write `mod math;` in `src/lib.rs`?
3. Why is everything `private` by default?
4. Make `fn helper` visible only within the current crate (not outside):
   ```rust
   fn helper() { /* ... */ }
   ```
5. What does a Cargo *workspace* give you that separate packages don't?
6. Convert these two `use` statements into one:
   ```rust
   use std::collections::HashMap;
   use std::collections::BTreeMap;
   ```

<details>
<summary>Answers</summary>

1. A *package* is a Cargo bundle (one `Cargo.toml`) containing one or more crates. A *crate* is the unit the compiler builds: a single library or binary.
2. `src/math.rs`, **or** `src/math/mod.rs`. Pick one consistently; modern style prefers `src/math.rs` plus `src/math/` subdir for children.
3. To make API boundaries explicit. You opt items into your public surface with `pub`; everything else can change freely without breaking users.
4. `pub(crate) fn helper() { /* ... */ }`
5. A workspace shares `Cargo.lock`, `target/`, and lets you build/test multiple crates together. It also resolves dependency versions consistently across members.
6. ```rust
   use std::collections::{HashMap, BTreeMap};
   ```
</details>
