# Code Style

Follow [Effective Go](https://go.dev/doc/effective_go), [Google Go Style Guide](https://google.github.io/styleguide/go/best-practices.html), and [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md).

## Formatting

- Use `gofmt` / `goimports` for all formatting - no exceptions
- Keep line length reasonable (~100 characters)

## Naming

- Use meaningful, descriptive names over brevity
- Avoid type encoding in names (`users` not `usersMap`)
- Short names for limited scope, longer names for wider scope
- Avoid generic package names (`utils`, `common`, `helpers`)
- Name packages for what they provide, not what they contain

## Code Organization

- Group imports: stdlib, external, internal (blank line separated)
- Export only what needs to be public; prefer unexported by default
- Prefer fewer, larger packages over many small ones
- Use `internal/` for implementation details
- Group code logically; one concept per file is not required
- Implement functions by invokation order if possible

## Interfaces

- Define interfaces where consumed, not where implemented
- Keep interfaces small (1-3 methods ideal)
- Pass interfaces as values, not pointers
- Verify interface compliance at compile time:
  ```go
  var _ Interface = (*Implementation)(nil)
  ```

## Error Handling

- Return errors, don't panic; reserve panic for truly unrecoverable situations
- Handle errors exactly once (don't log and return)
- Add context that the underlying error doesn't provide
- Use `%w` for wrapping when callers need `errors.Is`/`errors.As`
- Use sentinel errors or custom types for programmatic handling

## Concurrency

- Never start a goroutine without knowing when it stops
- Prefer unbuffered or size-1 channels
- Copy slices/maps at boundaries to prevent data races
- Use `sync.Mutex` zero value; embed as non-pointer field

## Performance

- Prefer `strconv` over `fmt` for conversions
- Pre-allocate slices/maps with known capacity
- Avoid repeated string-to-byte conversions

## Comments & Documentation

- Comment exported symbols (godoc format)
- Explain *why*, not *what* the code does
- Don't comment bad code - rewrite it

## Functions & API Design

- Use `context.Context` as first parameter for functions that do I/O
- Avoid multiple parameters of the same type in a row
- Return early with guard clauses (line-of-sight coding)
- Design APIs that are hard to misuse

## Testing

- Use table-driven tests for comprehensive test coverage
- Name test cases descriptively
- Test behavior, not implementation

---

## Pre-Review Checklist

Before submitting code for review, verify:

- [ ] `gofmt` / `goimports` applied
- [ ] No exported symbols without documentation
- [ ] Errors handled exactly once (not logged and returned)
- [ ] No goroutines without clear shutdown mechanism
- [ ] Interfaces defined at consumer, not provider
- [ ] No generic package names (`utils`, `common`, `helpers`)
- [ ] `context.Context` is first parameter for I/O functions
- [ ] No panics except for truly unrecoverable situations
- [ ] Slices/maps copied at API boundaries if shared
- [ ] Tests cover behavior, not implementation details
