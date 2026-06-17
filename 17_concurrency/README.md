# 17 — Fearless Concurrency

## Concept summary

Rust's ownership + type system rules out **data races** *at compile time*. You'll still see deadlocks, livelocks, and logic bugs — but no UB from shared mutable memory.

### Threads

```rust
use std::thread;
let h = thread::spawn(|| {
    println!("from a thread");
    42
});
let result = h.join().unwrap();         // wait + collect return value
```

Capture environment with `move`:

```rust
let v = vec![1, 2, 3];
thread::spawn(move || println!("{v:?}")).join().unwrap();
```

### `Send` & `Sync` (auto traits)

- `T: Send` — safe to **transfer ownership** to another thread.
- `T: Sync` — safe to **share `&T`** across threads (i.e. `&T: Send`).
- Most types are both. Notably **not** thread-safe: `Rc<T>`, `RefCell<T>`, raw pointers.

### Channels — message passing

Single-producer, single-consumer / multi-producer, single-consumer:

```rust
use std::sync::mpsc;
let (tx, rx) = mpsc::channel();

let tx2 = tx.clone();
thread::spawn(move || tx.send("hi from A").unwrap());
thread::spawn(move || tx2.send("hi from B").unwrap());

for msg in rx { println!("{msg}"); }    // iterates until all senders dropped
```

Sending **moves** the value across threads — no shared state, no races. (For bounded / multi-consumer channels see `crossbeam-channel` or `tokio::sync::mpsc`.)

### Shared state — `Mutex<T>` & `RwLock<T>` with `Arc<T>`

```rust
use std::sync::{Arc, Mutex};
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];
for _ in 0..10 {
    let c = Arc::clone(&counter);
    handles.push(thread::spawn(move || {
        *c.lock().unwrap() += 1;       // lock, mutate, drop guard
    }));
}
for h in handles { h.join().unwrap(); }
println!("{}", *counter.lock().unwrap());   // 10
```

- `Mutex::lock()` returns a `Result<MutexGuard<T>, _>`; `Err` only on **poisoning** (a thread panicked while holding the lock).
- The guard is a smart pointer; when it drops, the lock releases (RAII).

### Atomics

`std::sync::atomic::{AtomicBool, AtomicUsize, AtomicI32, ...}` — lock-free primitives. Pick the right memory ordering (`Relaxed`, `Acquire`, `Release`, `SeqCst`) — start with `SeqCst` if you're unsure.

### `thread::scope` (stable since 1.63)

Lets threads borrow non-`'static` data safely:

```rust
let v = vec![1, 2, 3];
std::thread::scope(|s| {
    s.spawn(|| println!("{v:?}"));      // can borrow v
});
```

## Quiz

1. What's the difference between `Send` and `Sync`?
2. Why is `Rc<T>` not `Send`, but `Arc<T>` is?
3. Predict — does this compile?
   ```rust
   use std::thread;
   let v = vec![1, 2, 3];
   let h = thread::spawn(|| println!("{v:?}"));
   h.join().unwrap();
   ```
4. Add the minimum change to make it compile.
5. Why is a `Mutex<T>` typically wrapped in `Arc<T>` when sharing across threads?
6. What does "lock poisoning" mean and when does it occur?

<details>
<summary>Answers</summary>

1. `Send` means the *value* can be moved to another thread. `Sync` means a *reference* `&T` can be shared across threads (equivalently: `T: Sync` iff `&T: Send`).
2. `Rc<T>`'s reference count is updated non-atomically — concurrent updates would race. `Arc<T>` uses atomic increments/decrements, so it's safe to clone across threads.
3. **No.** The closure captures `v` by *reference*, but the spawned thread requires `'static` data. The borrow checker rejects it.
4. Add `move`: `let h = thread::spawn(move || println!("{v:?}"));` — transfers ownership of `v` into the thread.
5. The `Mutex` itself isn't `Clone` and has a single home; we need multiple owners across threads. `Arc` provides shared ownership; the `Mutex` provides synchronized mutation. The combo is `Arc<Mutex<T>>`.
6. If a thread panics while holding a `Mutex`, the lock is *poisoned*. Subsequent `lock()` calls return `Err(PoisonError)` so callers know the protected state may be inconsistent. You can still recover it with `.into_inner()` if you choose to.
</details>
