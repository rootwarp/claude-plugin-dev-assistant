# /cargo-check

Quickly check Rust code for errors without producing binaries.

## Instructions

Run cargo check for fast compilation checking:

```bash
cargo check
```

This is faster than `cargo build` as it skips code generation.

To check all targets including tests and examples:

```bash
cargo check --all-targets
```

If there are errors, analyze them and suggest fixes.
