# Concurrency

## Ownership as Race Prevention

Rust's ownership system prevents data races at compile time. Understanding these guarantees is essential:

- **Ownership**: Only one owner at a time; no concurrent access by default
- **Borrowing**: Multiple `&T` (shared) OR one `&mut T` (exclusive), never both
- **Lifetimes**: Compiler ensures references don't outlive data

```rust
// Compile error: cannot borrow as mutable while borrowed as immutable
let mut data = vec![1, 2, 3];
let reference = &data[0];
data.push(4); // Error!
println!("{}", reference);
```

## Send and Sync Traits

These marker traits define thread safety:

- **`Send`**: Type can be transferred to another thread
- **`Sync`**: Type can be shared between threads (`&T` is `Send`)

### Common Non-Send Types

| Type | Why Not Send |
|------|--------------|
| `Rc<T>` | Non-atomic reference counting |
| `RefCell<T>` | Non-thread-safe interior mutability |
| `*const T`, `*mut T` | Raw pointers have no safety guarantees |
| Most iterators | May hold references to non-Send data |

```rust
use std::rc::Rc;
use std::sync::Arc;

// Compile error: Rc is not Send
let rc = Rc::new(5);
std::thread::spawn(move || println!("{}", rc)); // Error!

// Fix: use Arc for thread-safe reference counting
let arc = Arc::new(5);
std::thread::spawn(move || println!("{}", arc)); // OK
```

## Shared State Patterns

### Arc<Mutex<T>> for Shared Mutable State

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());
```

### RwLock for Read-Heavy Workloads

```rust
use std::sync::{Arc, RwLock};

let cache = Arc::new(RwLock::new(HashMap::new()));

// Multiple readers allowed
let cache_read = cache.read().unwrap();
let value = cache_read.get(&key);

// Exclusive write access
let mut cache_write = cache.write().unwrap();
cache_write.insert(key, value);
```

### Atomic Operations

Use atomics for simple shared state:

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;

let counter = Arc::new(AtomicUsize::new(0));

// Increment
counter.fetch_add(1, Ordering::SeqCst);

// Load
let value = counter.load(Ordering::SeqCst);
```

**Memory Ordering Guidelines**:
- `Ordering::SeqCst`: Safe default; sequential consistency
- `Ordering::Relaxed`: Only for independent counters/flags
- `Ordering::Acquire`/`Release`: For synchronization pairs
- When in doubt, use `SeqCst`

## Message Passing

### Standard Library mpsc

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel();

// Producer
thread::spawn(move || {
    for i in 0..10 {
        tx.send(i).unwrap();
    }
});

// Consumer
for received in rx {
    println!("Got: {}", received);
}
```

### Crossbeam Channels

Prefer `crossbeam-channel` for more features:

```rust
use crossbeam_channel::{bounded, select, unbounded};

// Bounded channel (backpressure)
let (tx, rx) = bounded::<i32>(10);

// Select across multiple channels
select! {
    recv(rx1) -> msg => handle_msg1(msg),
    recv(rx2) -> msg => handle_msg2(msg),
    default => println!("No message available"),
}
```

## Async Considerations

### Use tokio::sync for Async Code

Never use `std::sync::Mutex` across `.await` points:

```rust
// Bad: std::sync::Mutex held across await
async fn bad_example(mutex: Arc<std::sync::Mutex<Data>>) {
    let guard = mutex.lock().unwrap();
    some_async_operation().await; // Blocks other threads!
    drop(guard);
}

// Good: use tokio::sync::Mutex
async fn good_example(mutex: Arc<tokio::sync::Mutex<Data>>) {
    let guard = mutex.lock().await;
    some_async_operation().await;
    drop(guard);
}
```

### Async-Aware Primitives

```rust
use tokio::sync::{Mutex, RwLock, Semaphore, mpsc, oneshot};

// Async mutex
let mutex = Mutex::new(data);
let guard = mutex.lock().await;

// Async channel
let (tx, mut rx) = mpsc::channel(100);
tx.send(value).await?;

// One-shot for single response
let (tx, rx) = oneshot::channel();
tx.send(response)?;
let result = rx.await?;

// Semaphore for limiting concurrency
let semaphore = Arc::new(Semaphore::new(10));
let permit = semaphore.acquire().await?;
```

### spawn_blocking for CPU-Bound Work

```rust
use tokio::task;

async fn process_data(data: Vec<u8>) -> Result<Output> {
    // Move CPU-intensive work off the async runtime
    let result = task::spawn_blocking(move || {
        expensive_computation(&data)
    }).await?;

    Ok(result)
}
```

## Deadlock Patterns

### Lock Ordering

Always acquire locks in a consistent order:

```rust
// Bug: potential deadlock
// Thread 1: lock A, then B
// Thread 2: lock B, then A

// Fix: define a global lock order
fn transfer(from: &Account, to: &Account, amount: u64) {
    // Always lock lower ID first
    let (first, second) = if from.id < to.id {
        (from, to)
    } else {
        (to, from)
    };

    let _guard1 = first.balance.lock().unwrap();
    let _guard2 = second.balance.lock().unwrap();
    // ... perform transfer
}
```

### Self-Deadlock with Non-Reentrant Locks

```rust
// Bug: Mutex is not reentrant
fn outer(mutex: &Mutex<Data>) {
    let _guard = mutex.lock().unwrap();
    inner(mutex); // Deadlock!
}

fn inner(mutex: &Mutex<Data>) {
    let _guard = mutex.lock().unwrap(); // Blocks forever
}

// Fix: restructure to avoid nested locking, or use parking_lot::ReentrantMutex
```

### Async Deadlock with Blocking

```rust
// Bug: blocking in async context
async fn bad(mutex: Arc<std::sync::Mutex<Data>>) {
    let _guard = mutex.lock().unwrap(); // Blocks the runtime thread!
}

// Fix: use async-aware primitives or spawn_blocking
async fn good(mutex: Arc<tokio::sync::Mutex<Data>>) {
    let _guard = mutex.lock().await;
}
```

## Common Anti-Patterns

### Holding Locks Too Long

```rust
// Bad: lock held during I/O
fn update_and_save(cache: &Mutex<Cache>, key: &str) {
    let mut guard = cache.lock().unwrap();
    guard.insert(key, compute_value());
    save_to_disk(&guard); // Slow I/O while holding lock!
}

// Good: minimize lock scope
fn update_and_save(cache: &Mutex<Cache>, key: &str) {
    let value = compute_value();
    let snapshot = {
        let mut guard = cache.lock().unwrap();
        guard.insert(key, value);
        guard.clone()
    };
    save_to_disk(&snapshot);
}
```

### Arc<Mutex<Vec<T>>> for Append-Only

```rust
// Overkill for append-only log
let log = Arc::new(Mutex::new(Vec::new()));

// Better: use a channel or lock-free structure
let (tx, rx) = crossbeam_channel::unbounded();
tx.send(entry)?;
```

---

## Pre-Review Checklist

Before submitting concurrent code for review, verify:

- [ ] `Arc` used instead of `Rc` for shared ownership across threads
- [ ] `tokio::sync` primitives used in async code, not `std::sync`
- [ ] No `std::sync::Mutex` held across `.await` points
- [ ] CPU-bound work uses `spawn_blocking`
- [ ] Lock ordering is consistent to prevent deadlocks
- [ ] Lock scopes are minimized (no I/O while holding locks)
- [ ] Atomic operations use appropriate memory ordering (`SeqCst` if unsure)
- [ ] Channels preferred over shared state when possible
- [ ] No `unwrap()` on lock results in production code (handle poisoning)
- [ ] Thread-safety traits (`Send`/`Sync`) are satisfied
