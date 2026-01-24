# /python:init-dir

Initialize a standard Python project with uv and modern tooling.

## Instructions

### Step 1: Gather Information

Ask the user:
1. **Project type:**
   - Application (recommended) - executable with `src/` layout
   - Library - reusable package with `src/` layout
   - Simple Script - flat layout for small projects

2. **Project name** (used for package name and directory)

3. **Additional features:**
   - Web framework (FastAPI/Flask/none)
   - CLI framework (Typer/Click/none)
   - Database (SQLAlchemy/asyncpg/none)
   - Type checking (mypy/pyright)

### Step 2: Create Project with uv

```bash
uv init <project-name>
cd <project-name>
```

### Step 3: Create Directory Structure

**Application/Library (src layout):**
```bash
mkdir -p src/<package_name> tests docs
touch src/<package_name>/__init__.py
touch src/<package_name>/py.typed
touch tests/__init__.py
touch tests/conftest.py
```

**Simple Script:**
```bash
mkdir -p <package_name> tests
touch <package_name>/__init__.py
touch tests/__init__.py
```

### Step 4: Configure pyproject.toml

Update `pyproject.toml`:

```toml
[project]
name = "<project-name>"
version = "0.1.0"
description = "A brief description"
readme = "README.md"
license = { text = "MIT" }
requires-python = ">=3.11"
authors = [
    { name = "Your Name", email = "you@example.com" }
]
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=4.0",
    "ruff>=0.4",
    "mypy>=1.10",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/<package_name>"]

[tool.ruff]
line-length = 79
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM", "RUF"]

[tool.ruff.lint.isort]
known-first-party = ["<package_name>"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_ignores = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=<package_name>"
```

### Step 5: Add Selected Dependencies

Based on user choices, add to dependencies:

**FastAPI:**
```bash
uv add fastapi "uvicorn[standard]"
```

**Typer CLI:**
```bash
uv add typer
```

Add to pyproject.toml:
```toml
[project.scripts]
<app-name> = "<package_name>.cli:app"
```

**SQLAlchemy:**
```bash
uv add sqlalchemy alembic
```

### Step 6: Create Source Files

**`src/<package_name>/__init__.py`:**
```python
"""<project-name> - A brief description."""

__version__ = "0.1.0"
```

**`src/<package_name>/main.py` (for applications):**
```python
"""Main entry point."""


def main() -> None:
    """Run the application."""
    print("Hello, World!")


if __name__ == "__main__":
    main()
```

**`src/<package_name>/py.typed`:**
```
# Marker file for PEP 561
```

### Step 7: Create Test Files

**`tests/conftest.py`:**
```python
"""Pytest configuration and fixtures."""

import pytest


@pytest.fixture
def sample_data() -> dict[str, str]:
    """Provide sample test data."""
    return {"key": "value"}
```

**`tests/test_main.py`:**
```python
"""Tests for main module."""

from <package_name> import __version__


def test_version() -> None:
    """Test version is set."""
    assert __version__ == "0.1.0"
```

### Step 8: Write .gitignore

Create `.gitignore`:

```gitignore
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# Distribution / packaging
dist/
build/
*.egg-info/
*.egg

# Virtual environments
.venv/
venv/
ENV/

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Environment files
.env
.env.local
*.env

# Testing
.pytest_cache/
.coverage
htmlcov/
coverage.xml
*.cover

# Type checking
.mypy_cache/
.pyright/

# Ruff
.ruff_cache/
```

### Step 9: Write .python-version (optional)

Create `.python-version` for pyenv:

```
3.11
```

### Step 10: Install Dependencies

```bash
uv add --dev pytest pytest-cov ruff mypy
uv sync
```

### Step 11: Display Result

Show the created structure using `tree` or `ls -la` and confirm completion.

Run initial commands:
```bash
# Format code
uv run ruff format .

# Check linting
uv run ruff check .

# Type check
uv run mypy src/

# Run tests
uv run pytest
```

## Rules

- Use `src/` layout for packages to prevent import confusion
- Always include `py.typed` for typed packages
- Use `pyproject.toml` as single configuration file
- Python 3.11+ for modern features
- **Line length: 79 characters** (PEP 8 strict)
- Use `uv` for all dependency management
