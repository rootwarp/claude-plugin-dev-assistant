# /uv

Manage Python dependencies using uv.

## Instructions

### Sync Dependencies

Install all dependencies from `uv.lock`:

```bash
uv sync
```

### Add Dependencies

Add a runtime dependency:

```bash
uv add <package>
```

Add a dev dependency:

```bash
uv add --dev <package>
```

Add multiple packages:

```bash
uv add httpx pydantic sqlalchemy
```

Add with version constraint:

```bash
uv add "httpx>=0.27"
```

### Remove Dependencies

```bash
uv remove <package>
```

### Update Dependencies

Update all dependencies:

```bash
uv lock --upgrade
uv sync
```

Update a specific package:

```bash
uv lock --upgrade-package <package>
uv sync
```

### Run Commands

Run a command in the virtual environment:

```bash
uv run python -m <module>
uv run pytest
uv run ruff check .
```
