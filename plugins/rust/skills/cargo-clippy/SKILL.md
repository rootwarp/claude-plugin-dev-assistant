# /cargo-clippy

Run Clippy linter on the Rust project.

## Instructions

Run Clippy to check for common mistakes and improvements:

```bash
cargo clippy
```

For stricter checking (deny all warnings):

```bash
cargo clippy -- -D warnings
```

To fix automatically fixable issues:

```bash
cargo clippy --fix
```

Analyze any lint warnings or errors and suggest fixes.
