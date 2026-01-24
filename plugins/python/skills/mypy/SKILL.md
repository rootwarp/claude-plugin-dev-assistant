# /mypy

Run type checking on the Python project using uv.

## Instructions

Run mypy for type checking:

```bash
uv run mypy src/
```

Or check specific files:

```bash
uv run mypy src/<package_name>/main.py
```

For strict mode:

```bash
uv run mypy --strict src/
```

Common options:

```bash
# Show error codes
uv run mypy --show-error-codes src/

# Ignore missing imports
uv run mypy --ignore-missing-imports src/

# Generate HTML report
uv run mypy --html-report mypy-report src/
```

Analyze any type errors and suggest fixes with proper type annotations.
