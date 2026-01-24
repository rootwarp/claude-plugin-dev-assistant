# /cargo-doc

Generate and view Rust documentation.

## Instructions

Generate documentation for the project and dependencies:

```bash
cargo doc
```

To open documentation in browser:

```bash
cargo doc --open
```

To generate documentation without dependencies:

```bash
cargo doc --no-deps
```

## Looking Up Crate Documentation

For crate documentation, use docs.rs:

### Arguments

- `crate`: The crate name to look up (e.g., `tokio`, `serde`, `axum`)
- `version` (optional): Specific version (e.g., `1.0.0`). Defaults to latest.
- `item` (optional): Specific function, type, or module to focus on

### URL Structure

| Type | URL Format | Example |
|------|------------|---------|
| Latest | `https://docs.rs/{crate}` | `https://docs.rs/tokio` |
| Specific version | `https://docs.rs/{crate}/{version}` | `https://docs.rs/tokio/1.36.0` |
| Specific item | `https://docs.rs/{crate}/latest/{crate}/{path}` | `https://docs.rs/tokio/latest/tokio/sync/index.html` |

### Execution

When invoked with a crate name, use WebFetch to retrieve the documentation:

```
WebFetch url="https://docs.rs/{crate}" prompt="Extract the documentation for this Rust crate. Include: 1) Crate overview 2) Key modules and their purposes 3) Important types and traits 4) Usage examples. If a specific item was requested, focus on that item's documentation."
```

## Standard Library Documentation

For standard library docs, use:

- Base URL: `https://doc.rust-lang.org/std/`
- Example: `https://doc.rust-lang.org/std/collections/struct.HashMap.html`

```
WebFetch url="https://doc.rust-lang.org/std/{path}" prompt="Extract the documentation for this Rust standard library item. Include its signature, description, methods, and examples."
```
