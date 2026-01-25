# Code Style

Follow [The Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/), [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/), and [Clippy Lints](https://rust-lang.github.io/rust-clippy/master/).

## Formatting

- Use `rustfmt` for all formatting - no exceptions
- Keep line length at 100 characters (rustfmt default)
- Use 4 spaces for indentation

## Naming

- Types: `PascalCase` (structs, enums, traits, type aliases)
- Functions/methods: `snake_case`
- Variables/fields: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Modules: `snake_case`
- Crates: `snake_case` (prefer single words or hyphenated)
- Lifetimes: short lowercase (`'a`, `'b`, `'ctx`)
- Type parameters: single uppercase letters (`T`, `E`, `K`, `V`) or descriptive (`Item`, `Error`)

## Code Organization

- Group imports: std, external crates, crate-internal (blank line separated)
- Order within modules: types, implementations, functions
- Keep modules focused; one concept per module
- Use `pub(crate)` for internal visibility; avoid `pub` by default
- Re-export from `lib.rs` for clean public API

```rust
// Good import organization
use std::collections::HashMap;
use std::io::{self, Read, Write};

use serde::{Deserialize, Serialize};
use tokio::sync::mpsc;

use crate::config::Config;
use crate::error::Result;
```

## Traits

- Keep traits small and focused (1-5 methods ideal)
- Use default implementations where sensible
- Derive common traits when appropriate: `Debug`, `Clone`, `PartialEq`
- Prefer composition over trait inheritance
- Use extension traits for adding methods to foreign types

```rust
// Good: focused trait
pub trait Reader {
    fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>;
}

// Derive common traits
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct UserId(String);
```

## Error Handling

- Use `Result<T, E>` for recoverable errors; never panic for expected failures
- Define custom error types with `thiserror` or manual implementation
- Use `?` operator for error propagation
- Add context to errors using `anyhow` or custom error types
- Avoid `.unwrap()` and `.expect()` in library code
- Use `Option` for optional values, not sentinel values

```rust
// Good: custom error with context
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("failed to read config file: {path}")]
    Read { path: PathBuf, source: io::Error },

    #[error("invalid config format")]
    Parse(#[from] toml::de::Error),
}

// Good: error propagation with context
fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = fs::read_to_string(path)
        .map_err(|e| ConfigError::Read { path: path.to_owned(), source: e })?;
    let config: Config = toml::from_str(&content)?;
    Ok(config)
}
```

## Ownership and Borrowing

- Prefer borrowing over cloning when possible
- Use `&str` over `String` in function parameters
- Return owned types from constructors and factories
- Use `Cow<'_, str>` when ownership is conditional
- Clone explicitly; avoid hidden copies

```rust
// Good: borrow in parameters, own in return
fn process_name(name: &str) -> String {
    name.trim().to_uppercase()
}

// Good: Cow for conditional ownership
fn normalize(input: &str) -> Cow<'_, str> {
    if input.contains(' ') {
        Cow::Owned(input.replace(' ', "_"))
    } else {
        Cow::Borrowed(input)
    }
}
```

## Concurrency

See `concurrency.md` for comprehensive coverage of Send/Sync traits, shared state patterns, message passing, async considerations, and deadlock prevention.

## Performance

- Avoid unnecessary allocations; reuse buffers
- Use iterators over indexed loops
- Prefer `&[T]` over `&Vec<T>` in function signatures
- Use `Box<[T]>` for fixed-size heap allocations
- Profile before optimizing; use `#[inline]` sparingly

## Comments & Documentation

- Document all public items with `///` doc comments
- Include examples in documentation using ```` ```rust ````
- Use `//!` for module-level documentation
- Explain *why*, not *what*; code should be self-explanatory
- Add `# Panics`, `# Errors`, `# Safety` sections where applicable

```rust
/// Parses a configuration file from the given path.
///
/// # Errors
///
/// Returns an error if the file cannot be read or contains invalid TOML.
///
/// # Examples
///
/// ```
/// let config = parse_config("config.toml")?;
/// ```
pub fn parse_config(path: &str) -> Result<Config, ConfigError> {
    // ...
}
```

## Testing

- Place unit tests in `#[cfg(test)]` module within the same file
- Use descriptive test names: `test_function_scenario_expected_behavior`
- Use `#[should_panic]` for testing panic conditions
- Mock external dependencies using traits
- Integration tests go in `tests/` directory

---

## MANDATORY: Quality Gate Before Commits

**These steps MUST be run and pass before any commit or PR. Do NOT skip these steps.**

### 1. Format Code (REQUIRED)

```bash
cargo fmt --all
```

- Run this after every code change
- Fix any formatting issues before proceeding
- Never commit unformatted code

### 2. Run Clippy Lints (REQUIRED)

```bash
cargo clippy --all-targets --all-features -- -D warnings
```

- All warnings must be resolved (not suppressed without justification)
- Address each lint, don't just `#[allow(...)]` them
- If a lint must be suppressed, add a comment explaining why

### 3. Run Tests (REQUIRED)

```bash
cargo test --all-features
```

- All tests must pass
- Do NOT proceed if any test fails
- Add tests for new functionality

### 4. Verify Before Commit

Run all checks together:

```bash
cargo fmt --all --check && cargo clippy --all-targets --all-features -- -D warnings && cargo test --all-features
```

**If any of these commands fail, fix the issues before committing.**

---

## Pre-Review Checklist

Before submitting code for review, verify:

- [ ] `cargo fmt --all` applied - **code is formatted**
- [ ] `cargo clippy` passes with **zero warnings**
- [ ] `cargo test` passes with **all tests green**
- [ ] No `.unwrap()` or `.expect()` in library code
- [ ] All public items have documentation
- [ ] Errors include meaningful context
- [ ] No unnecessary clones or allocations
- [ ] Tests cover main functionality
- [ ] No `unsafe` without safety documentation
