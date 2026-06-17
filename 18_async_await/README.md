# 18 — Async / Await

## Concept summary

`async`/`await` lets a single OS thread juggle many in-flight tasks (network calls, timers, file I/O) without blocking. It's *not* parallelism by itself — it's *concurrency*.

### Futures

- An `async fn` or `async { ... }` block returns a value that implements `Future<Output = T>`.
- Futures are **lazy**: they do nothing until polled by an executor.
- `.await` polls a future and yields control if it isn't ready.

```rust
async fn fetch(url: &str) -> String { /* ... */ }

async fn run() {
    let body = fetch("https://example.com").await;
    println!("{body}");
}
```

### You need an executor

The standard library defines the `Future` trait but ships **no executor**. Choose a runtime:

- **`tokio`** — the de-facto standard, multi-threaded, full ecosystem.
- **`async-std`** — std-like API, single binary.
- **`smol`** — small embeddable executor.

```rust
#[tokio::main]
async fn main() {
    let body = fetch("https://example.com").await;
    println!("{body}");
}
```

### Spawning & joining

```rust
use tokio::task;
let h1 = task::spawn(async { do_work().await });
let h2 = task::spawn(async { do_other().await });
let (a, b) = tokio::join!(h1, h2);          // wait for both
```

- `tokio::join!` runs concurrently in the **same task**.
- `task::spawn` puts work onto another task (possibly another thread).
- `tokio::select!` waits for the **first** of several futures to complete.

### Blocking vs. non-blocking

- Never call blocking I/O (`std::fs`, `std::net`) inside `async` — use the runtime's async equivalents (`tokio::fs`, `tokio::net`).
- For CPU-heavy work, offload via `task::spawn_blocking`.

### Common pitfalls

- Forgetting `.await` makes a future a no-op (compiler warns).
- `?` inside `async fn` works just like sync code if the function returns `Result`.
- Sharing state: same as threads — `Arc<Mutex<T>>` or `tokio::sync::Mutex` (the latter doesn't block the executor under contention).
- `Send` bounds are still required for tasks moved across the executor's threads — `Rc`/`RefCell` won't work; use `Arc`/`tokio::sync::*`.

## Quiz

1. Does this code do any work?
   ```rust
   fn main() {
       async { println!("hi"); };
   }
   ```
2. What's the difference between `tokio::join!` and `tokio::task::spawn`?
3. When should you use `spawn_blocking`?
4. Why can't you use `std::sync::Mutex` *across* `.await` points without care, and what's the alternative?
5. Sketch an `async fn` that fetches two URLs concurrently and returns both bodies.
6. What does `.await` actually do behind the scenes (one-sentence model)?

<details>
<summary>Answers</summary>

1. **No.** Futures are lazy — the `async { ... }` block builds a future and immediately drops it without polling. Nothing prints. You need an executor and an `.await` (or `spawn`).
2. `join!` runs the supplied futures concurrently on the **current task** and waits for all to finish. `spawn` schedules a new **independent task** (possibly on another thread) and returns a handle that itself is a future.
3. When you need to run CPU-bound or genuinely blocking sync code without stalling the async runtime's worker threads. `spawn_blocking` puts the work on a dedicated blocking thread pool.
4. Holding a `std::sync::Mutex` guard across an `.await` blocks the executor thread if another task awaits the lock — and the held guard is `!Send`, so the task can't migrate. Use `tokio::sync::Mutex`, which yields cooperatively instead of blocking the OS thread.
5. ```rust
   async fn two_bodies(a: &str, b: &str) -> (String, String) {
       let (ra, rb) = tokio::join!(fetch(a), fetch(b));
       (ra, rb)
   }
   ```
6. `.await` polls the future; if it returns `Poll::Pending` the current task yields back to the executor (registering a waker), and resumes here when the future signals readiness.
</details>
