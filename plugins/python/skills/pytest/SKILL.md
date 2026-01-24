# /pytest

Run tests for the Python project using uv.

## Instructions

Run tests with pytest:

```bash
uv run pytest
```

For verbose output:

```bash
uv run pytest -v
```

With coverage:

```bash
uv run pytest --cov=<package_name> --cov-report=html
```

To run a specific test file or function:

```bash
uv run pytest tests/test_main.py
uv run pytest tests/test_main.py::test_function_name
```

To see print output:

```bash
uv run pytest -s
```

If tests fail, analyze the failures and suggest fixes.
