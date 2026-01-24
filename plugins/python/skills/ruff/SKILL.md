# /ruff

Run Ruff linter and formatter on the Python project using uv.

## Instructions

### Linting

Check for lint issues:

```bash
uv run ruff check .
```

Auto-fix issues where possible:

```bash
uv run ruff check --fix .
```

### Formatting

Format all Python files (79 char line limit per PEP 8):

```bash
uv run ruff format .
```

Check formatting without making changes:

```bash
uv run ruff format --check .
```

### Combined

Run both check and format:

```bash
uv run ruff check --fix . && uv run ruff format .
```

Analyze any lint warnings or errors and suggest fixes.
