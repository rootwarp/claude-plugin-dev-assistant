# Project Layout

Follow [Python Packaging User Guide](https://packaging.python.org/) conventions and modern best practices.

## Standard Project Layout

### src Layout (Recommended)

```
project/
├── pyproject.toml        # Project configuration
├── README.md
├── LICENSE
├── src/
│   └── mypackage/        # Package code
│       ├── __init__.py
│       ├── main.py
│       └── utils.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py       # pytest fixtures
│   └── test_main.py
└── docs/
```

### Flat Layout (Simple projects)

```
project/
├── pyproject.toml
├── README.md
├── mypackage/
│   ├── __init__.py
│   └── main.py
└── tests/
    └── test_main.py
```

## Directory Purposes

| Directory | Purpose |
|-----------|---------|
| `src/<package>/` | Source code (src layout) |
| `tests/` | Test files |
| `docs/` | Documentation |
| `scripts/` | Utility scripts |
| `data/` | Data files (not for secrets) |
| `.github/` | GitHub Actions workflows |

## pyproject.toml Configuration

Modern Python projects use `pyproject.toml` as the single configuration file:

```toml
[project]
name = "mypackage"
version = "0.1.0"
description = "A brief description"
readme = "README.md"
license = { text = "MIT" }
requires-python = ">=3.11"
authors = [
    { name = "Your Name", email = "you@example.com" }
]
keywords = ["keyword1", "keyword2"]
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]
dependencies = [
    "httpx>=0.27",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=4.0",
    "ruff>=0.4",
    "mypy>=1.10",
]

[project.scripts]
myapp = "mypackage.cli:main"

[project.urls]
Homepage = "https://github.com/username/mypackage"
Documentation = "https://mypackage.readthedocs.io"
Repository = "https://github.com/username/mypackage"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/mypackage"]

[tool.ruff]
line-length = 79
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM"]

[tool.mypy]
python_version = "3.11"
strict = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=mypackage"
```

## Package Manager: uv

Use [uv](https://docs.astral.sh/uv/) for fast, reliable dependency management.

```bash
# Create new project
uv init myproject
cd myproject

# Add dependencies
uv add httpx pydantic

# Add dev dependencies
uv add --dev pytest ruff mypy

# Sync dependencies
uv sync

# Run commands
uv run python -m mypackage
uv run pytest

# Update dependencies
uv lock --upgrade
```

## Module Organization

```python
# src/mypackage/__init__.py
"""MyPackage - A brief description."""

from mypackage.core import process
from mypackage.models import Config, User

__version__ = "0.1.0"
__all__ = ["process", "Config", "User"]
```

```python
# src/mypackage/core.py
"""Core processing functions."""

from mypackage.models import Config

def process(config: Config) -> None:
    """Process data according to configuration."""
    ...
```

## Rules

- Use `src/` layout for libraries to prevent import confusion
- Flat layout is acceptable for simple applications
- Always include `py.typed` marker for typed packages
- Keep `__init__.py` minimal; use for re-exports
- One logical concept per module
- Tests mirror source structure: `tests/test_<module>.py`

## Recommended Starting Structure

```
myproject/
├── pyproject.toml
├── README.md
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── py.typed        # Marker for typed package
│       └── main.py
└── tests/
    ├── __init__.py
    └── test_main.py
```

Add directories as needed, not preemptively.
