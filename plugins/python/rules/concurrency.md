# Concurrency

## GIL Implications

Python's Global Interpreter Lock (GIL) has significant implications:

- **Threading helps for**: I/O-bound tasks (network, disk, sleep)
- **Threading doesn't help for**: CPU-bound tasks (computation)
- **Use multiprocessing for**: CPU-bound parallelism

```python
# Good for I/O-bound: threading
import threading

def fetch_url(url: str) -> bytes:
    return requests.get(url).content

threads = [threading.Thread(target=fetch_url, args=(url,)) for url in urls]
for t in threads:
    t.start()
for t in threads:
    t.join()

# Good for CPU-bound: multiprocessing
from multiprocessing import Pool

def compute(x: int) -> int:
    return expensive_calculation(x)

with Pool() as pool:
    results = pool.map(compute, range(1000))
```

## Threading Primitives

### Lock (Mutex)

```python
import threading

class Counter:
    def __init__(self) -> None:
        self._value = 0
        self._lock = threading.Lock()

    def increment(self) -> None:
        with self._lock:
            self._value += 1

    def value(self) -> int:
        with self._lock:
            return self._value
```

### RLock (Reentrant Lock)

Use when the same thread may need to acquire the lock multiple times:

```python
import threading

class RecursiveCounter:
    def __init__(self) -> None:
        self._value = 0
        self._lock = threading.RLock()

    def increment(self) -> None:
        with self._lock:
            self._value += 1

    def increment_twice(self) -> None:
        with self._lock:
            self.increment()  # Can acquire lock again
            self.increment()
```

### Semaphore for Limiting Concurrency

```python
import threading

# Limit concurrent connections
connection_semaphore = threading.Semaphore(10)

def make_request(url: str) -> Response:
    with connection_semaphore:
        return requests.get(url)
```

### Event for Signaling

```python
import threading

shutdown_event = threading.Event()

def worker() -> None:
    while not shutdown_event.is_set():
        process_work()
        shutdown_event.wait(timeout=1.0)

# Signal shutdown
shutdown_event.set()
```

### Condition for Complex Coordination

```python
import threading
from collections import deque

class BoundedQueue:
    def __init__(self, maxsize: int) -> None:
        self._queue: deque = deque()
        self._maxsize = maxsize
        self._condition = threading.Condition()

    def put(self, item: object) -> None:
        with self._condition:
            while len(self._queue) >= self._maxsize:
                self._condition.wait()
            self._queue.append(item)
            self._condition.notify()

    def get(self) -> object:
        with self._condition:
            while not self._queue:
                self._condition.wait()
            item = self._queue.popleft()
            self._condition.notify()
            return item
```

## Thread-Safe Structures

### queue.Queue

The standard library provides thread-safe queues:

```python
import queue
import threading

work_queue: queue.Queue[Task] = queue.Queue()

def worker() -> None:
    while True:
        task = work_queue.get()
        if task is None:  # Shutdown signal
            break
        process(task)
        work_queue.task_done()

# Start workers
threads = [threading.Thread(target=worker) for _ in range(4)]
for t in threads:
    t.start()

# Add work
for task in tasks:
    work_queue.put(task)

# Wait for completion
work_queue.join()

# Shutdown workers
for _ in threads:
    work_queue.put(None)
for t in threads:
    t.join()
```

### Protected Collections Pattern

```python
import threading
from typing import TypeVar, Generic

K = TypeVar("K")
V = TypeVar("V")

class ThreadSafeDict(Generic[K, V]):
    def __init__(self) -> None:
        self._data: dict[K, V] = {}
        self._lock = threading.RLock()

    def get(self, key: K, default: V | None = None) -> V | None:
        with self._lock:
            return self._data.get(key, default)

    def set(self, key: K, value: V) -> None:
        with self._lock:
            self._data[key] = value

    def setdefault(self, key: K, value: V) -> V:
        with self._lock:
            return self._data.setdefault(key, value)
```

## Asyncio Race Conditions

### Coroutine Interleaving

Race conditions occur at `await` points, not between them:

```python
import asyncio

# Bug: race condition between await points
class AsyncCounter:
    def __init__(self) -> None:
        self.value = 0

    async def increment(self) -> None:
        current = self.value
        await asyncio.sleep(0)  # Yields control!
        self.value = current + 1  # Race condition

# Fix: use asyncio.Lock
class SafeAsyncCounter:
    def __init__(self) -> None:
        self.value = 0
        self._lock = asyncio.Lock()

    async def increment(self) -> None:
        async with self._lock:
            current = self.value
            await asyncio.sleep(0)
            self.value = current + 1
```

### Shared State in Tasks

```python
import asyncio

# Bug: shared mutable state
results: list[int] = []

async def fetch_and_append(url: str) -> None:
    result = await fetch(url)
    results.append(result)  # Not atomic!

# Fix: collect results from tasks
async def fetch_all(urls: list[str]) -> list[int]:
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(url)) for url in urls]
    return [task.result() for task in tasks]
```

### Asyncio Primitives

```python
import asyncio

# Lock
lock = asyncio.Lock()
async with lock:
    await critical_section()

# Semaphore
semaphore = asyncio.Semaphore(10)
async with semaphore:
    await limited_operation()

# Event
event = asyncio.Event()
await event.wait()
event.set()

# Queue
queue: asyncio.Queue[Task] = asyncio.Queue()
await queue.put(task)
task = await queue.get()
```

## concurrent.futures

### ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch(url: str) -> Response:
    return requests.get(url)

with ThreadPoolExecutor(max_workers=10) as executor:
    # Submit all tasks
    future_to_url = {
        executor.submit(fetch, url): url for url in urls
    }

    # Process as completed
    for future in as_completed(future_to_url):
        url = future_to_url[future]
        try:
            response = future.result()
        except Exception as e:
            print(f"Error fetching {url}: {e}")
```

### ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor

def compute(x: int) -> int:
    return expensive_calculation(x)

# Note: functions must be picklable (defined at module level)
with ProcessPoolExecutor() as executor:
    results = list(executor.map(compute, range(1000)))
```

### Context Manager Pattern

```python
from concurrent.futures import ThreadPoolExecutor
from contextlib import contextmanager
from typing import Iterator

@contextmanager
def managed_executor(max_workers: int = 4) -> Iterator[ThreadPoolExecutor]:
    executor = ThreadPoolExecutor(max_workers=max_workers)
    try:
        yield executor
    finally:
        executor.shutdown(wait=True)
```

## Common Anti-Patterns

### Check-Then-Act Race

```python
# Bug: race between check and act
if key not in cache:
    cache[key] = compute()  # Another thread may have set it

# Fix: use setdefault or lock
with cache_lock:
    if key not in cache:
        cache[key] = compute()

# Or use atomic operation
value = cache.setdefault(key, compute())  # Still calls compute()

# Best: use locking with double-check
with cache_lock:
    if key not in cache:
        cache[key] = compute()
    return cache[key]
```

### Lazy Initialization Race

```python
# Bug: race in lazy initialization
class Service:
    _instance = None

    @classmethod
    def get_instance(cls) -> "Service":
        if cls._instance is None:
            cls._instance = cls()  # Race condition!
        return cls._instance

# Fix: use threading.Lock
class Service:
    _instance = None
    _lock = threading.Lock()

    @classmethod
    def get_instance(cls) -> "Service":
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:  # Double-check
                    cls._instance = cls()
        return cls._instance
```

### Forgetting Thread-Safety in Async Code

```python
# Bug: using threading.Lock in async code
class BadAsyncCache:
    def __init__(self) -> None:
        self._lock = threading.Lock()  # Wrong!

    async def get(self, key: str) -> str:
        with self._lock:  # Blocks the event loop!
            return await self._fetch(key)

# Fix: use asyncio.Lock
class GoodAsyncCache:
    def __init__(self) -> None:
        self._lock = asyncio.Lock()

    async def get(self, key: str) -> str:
        async with self._lock:
            return await self._fetch(key)
```

### Blocking Calls in Async Functions

```python
import asyncio

# Bug: blocking call in async context
async def bad_read_file(path: str) -> str:
    return open(path).read()  # Blocks event loop!

# Fix: use run_in_executor for blocking I/O
async def good_read_file(path: str) -> str:
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(None, lambda: open(path).read())

# Better: use aiofiles
import aiofiles

async def best_read_file(path: str) -> str:
    async with aiofiles.open(path) as f:
        return await f.read()
```

---

## Pre-Review Checklist

Before submitting concurrent code for review, verify:

- [ ] Appropriate concurrency model chosen (threading for I/O, multiprocessing for CPU)
- [ ] All shared mutable state protected by locks
- [ ] `threading.Lock` used for threading, `asyncio.Lock` for asyncio
- [ ] No blocking calls in async functions (use `run_in_executor` if needed)
- [ ] `queue.Queue` used for thread communication (not bare lists)
- [ ] Thread/process pools properly shut down (use context managers)
- [ ] No check-then-act patterns without proper locking
- [ ] Lazy initialization uses double-checked locking
- [ ] `asyncio.TaskGroup` used for concurrent async tasks (Python 3.11+)
- [ ] Worker threads/processes have clean shutdown mechanism
