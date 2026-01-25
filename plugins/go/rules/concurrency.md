# Concurrency

## Race Detection

Always test concurrent code with the race detector:

```bash
# Run tests with race detection
go test -race ./...

# Build with race detection for development
go build -race

# Run specific benchmarks with race detection
go test -race -bench=. ./...
```

**CI Integration**: Enable `-race` flag in CI pipelines for all test runs. Race conditions caught in CI are far cheaper than production bugs.

## Goroutine Lifecycle

### Never Start a Goroutine Without a Shutdown Plan

Every goroutine must have a clear mechanism for termination.

```go
// Bad: goroutine leak - no way to stop
func startWorker() {
    go func() {
        for {
            process()
        }
    }()
}

// Good: context-based cancellation
func startWorker(ctx context.Context) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            default:
                process()
            }
        }
    }()
}
```

### WaitGroup for Goroutine Completion

```go
func processItems(items []Item) {
    var wg sync.WaitGroup

    for _, item := range items {
        wg.Add(1)
        go func(it Item) {
            defer wg.Done()
            process(it)
        }(item) // Pass item as argument to avoid capture bug
    }

    wg.Wait()
}
```

### Worker Pool Pattern

```go
func workerPool(ctx context.Context, jobs <-chan Job, numWorkers int) {
    var wg sync.WaitGroup

    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for {
                select {
                case job, ok := <-jobs:
                    if !ok {
                        return
                    }
                    process(job)
                case <-ctx.Done():
                    return
                }
            }
        }()
    }

    wg.Wait()
}
```

## Channel Patterns

### Close Semantics

- Only the sender should close a channel
- Never close a channel from the receiver side
- Closing a nil channel panics
- Sending on a closed channel panics
- Receiving from a closed channel returns zero value immediately

```go
// Good: sender closes
func producer(out chan<- int) {
    defer close(out)
    for i := 0; i < 10; i++ {
        out <- i
    }
}

// Good: range over channel (stops when closed)
func consumer(in <-chan int) {
    for val := range in {
        process(val)
    }
}
```

### Select for Multiple Channels

```go
func multiplex(ctx context.Context, a, b <-chan int, out chan<- int) {
    for {
        select {
        case v, ok := <-a:
            if !ok {
                a = nil // Disable this case
                continue
            }
            out <- v
        case v, ok := <-b:
            if !ok {
                b = nil
                continue
            }
            out <- v
        case <-ctx.Done():
            return
        }

        if a == nil && b == nil {
            return
        }
    }
}
```

### Channel Sizing Guidelines

- **Unbuffered (size 0)**: Synchronization required; sender blocks until receiver ready
- **Size 1**: Signal channels, single handoff
- **Known bounded size**: When producer/consumer rates are predictable
- **Avoid large buffers**: They hide problems; prefer backpressure

```go
// Signal channel (size 1)
done := make(chan struct{}, 1)

// Work queue with bounded concurrency
jobs := make(chan Job, numWorkers)
```

## Sync Package

### Mutex Best Practices

```go
type SafeCounter struct {
    mu    sync.Mutex // Zero value is valid, unlocked mutex
    count int
}

func (c *SafeCounter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

### RWMutex for Read-Heavy Workloads

```go
type Cache struct {
    mu    sync.RWMutex
    items map[string]Item
}

func (c *Cache) Get(key string) (Item, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    item, ok := c.items[key]
    return item, ok
}

func (c *Cache) Set(key string, item Item) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.items[key] = item
}
```

### sync.Once for One-Time Initialization

```go
var (
    instance *Config
    once     sync.Once
)

func GetConfig() *Config {
    once.Do(func() {
        instance = loadConfig()
    })
    return instance
}
```

### Atomic Operations

Use `sync/atomic` for simple counters and flags:

```go
type Stats struct {
    requests atomic.Int64
    errors   atomic.Int64
}

func (s *Stats) RecordRequest() {
    s.requests.Add(1)
}

func (s *Stats) RecordError() {
    s.errors.Add(1)
}
```

## Common Anti-Patterns

### Loop Variable Capture (Pre-Go 1.22)

```go
// Bug: all goroutines share the same loop variable
for _, item := range items {
    go func() {
        process(item) // Race condition!
    }()
}

// Fix: pass as argument
for _, item := range items {
    go func(it Item) {
        process(it)
    }(item)
}

// Note: Go 1.22+ fixes this with per-iteration loop variables
```

### Shared Map/Slice Without Synchronization

```go
// Bug: concurrent map write
var cache = make(map[string]int)

go func() { cache["a"] = 1 }()
go func() { cache["b"] = 2 }() // Panic: concurrent map writes

// Fix: use sync.Map or mutex
var cache sync.Map

go func() { cache.Store("a", 1) }()
go func() { cache.Store("b", 2) }()
```

### Check-Then-Act Race

```go
// Bug: race between check and act
if _, ok := cache[key]; !ok {
    cache[key] = compute() // Another goroutine may have set it
}

// Fix: use sync.Map's LoadOrStore
actual, loaded := cache.LoadOrStore(key, compute())
```

### Forgetting to Copy Slices at Boundaries

```go
type Server struct {
    mu      sync.Mutex
    clients []Client
}

// Bug: returns internal slice
func (s *Server) GetClients() []Client {
    s.mu.Lock()
    defer s.mu.Unlock()
    return s.clients // Caller can modify!
}

// Fix: return a copy
func (s *Server) GetClients() []Client {
    s.mu.Lock()
    defer s.mu.Unlock()
    result := make([]Client, len(s.clients))
    copy(result, s.clients)
    return result
}
```

---

## Pre-Review Checklist

Before submitting concurrent code for review, verify:

- [ ] All tests pass with `-race` flag
- [ ] Every goroutine has a clear shutdown mechanism (context, done channel, or WaitGroup)
- [ ] Channels are closed only by senders
- [ ] No shared maps/slices accessed without synchronization
- [ ] Loop variables are not captured in goroutines (or using Go 1.22+)
- [ ] Mutex/RWMutex used appropriately (RWMutex only for read-heavy workloads)
- [ ] No check-then-act patterns without proper locking
- [ ] Slices/maps copied at API boundaries
- [ ] Channel buffer sizes are justified (prefer unbuffered or size-1)
- [ ] `sync.Once` used for lazy initialization of singletons
