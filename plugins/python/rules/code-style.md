# Code Style

Follow [PEP 8](https://peps.python.org/pep-0008/) strictly, [PEP 257](https://peps.python.org/pep-0257/), and [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html).

## Formatting

- Use `ruff format` for all formatting - no exceptions
- **Line length: 79 characters maximum** (PEP 8 standard)
- Use 4 spaces for indentation
- Use `ruff` for import sorting

## Naming

- Classes: `PascalCase`
- Functions/methods: `snake_case`
- Variables: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Modules: `snake_case` (short, lowercase)
- Private: prefix with `_` (single underscore)
- Name mangling: prefix with `__` (double underscore, use sparingly)
- Type variables: `PascalCase` (`T`, `K`, `V`, `ItemT`)

## Code Organization

- Group imports: stdlib, third-party, local (blank line separated)
- Order imports alphabetically within groups
- Prefer absolute imports over relative imports
- One class per file for large classes; group related small classes
- Keep modules focused; split when exceeding ~500 lines

```python
# Good import organization
import os
import sys
from collections.abc import Callable, Iterator
from typing import Any

import httpx
from pydantic import BaseModel

from myapp.config import Settings
from myapp.models import User
```

## Type Hints

- Use type hints for all public functions and methods
- Use `from __future__ import annotations` for forward references
- Prefer built-in generics: `list[str]` over `List[str]` (Python 3.9+)
- Use `|` for unions: `str | None` over `Optional[str]` (Python 3.10+)
- Use `TypeAlias` for complex type definitions
- Run `mypy` or `pyright` for type checking

```python
from __future__ import annotations

from collections.abc import Callable, Sequence
from typing import TypeAlias

JsonValue: TypeAlias = str | int | float | bool | None | list["JsonValue"] | dict[str, "JsonValue"]

def process_items(
    items: Sequence[str],
    transform: Callable[[str], str] | None = None,
) -> list[str]:
    """Process items with optional transformation."""
    if transform is None:
        return list(items)
    return [transform(item) for item in items]
```

## Classes and Dataclasses

- Prefer `dataclass` or `pydantic.BaseModel` for data containers
- Use `@property` for computed attributes
- Prefer composition over inheritance
- Use `__slots__` for memory-critical classes

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass(frozen=True)
class User:
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)

    @property
    def domain(self) -> str:
        return self.email.split("@")[1]
```

## Error Handling

- Use specific exceptions, not bare `except:`
- Create custom exceptions for domain errors
- Don't catch exceptions you can't handle
- Use context managers for resource cleanup
- Prefer `raise ... from e` for exception chaining

```python
# Bad
try:
    do_something()
except:  # Too broad
    pass

# Good
class ConfigError(Exception):
    """Configuration-related errors."""

try:
    config = load_config(path)
except FileNotFoundError as e:
    raise ConfigError(f"Config file not found: {path}") from e
except json.JSONDecodeError as e:
    raise ConfigError(f"Invalid JSON in config: {path}") from e
```

## Functions

- Keep functions small and focused
- Use keyword-only arguments for clarity: `def func(*, name: str)`
- Return early with guard clauses
- Avoid mutable default arguments
- Use `*args` and `**kwargs` sparingly

```python
# Bad: mutable default
def add_item(item: str, items: list[str] = []) -> list[str]:
    items.append(item)
    return items

# Good: None default with initialization
def add_item(item: str, items: list[str] | None = None) -> list[str]:
    if items is None:
        items = []
    items.append(item)
    return items
```

## Concurrency

See `concurrency.md` for comprehensive coverage of GIL implications, threading primitives, asyncio race conditions, concurrent.futures patterns, and thread-safe data structures.

## Comments & Documentation

- Use docstrings for all public modules, classes, and functions
- Follow Google or NumPy docstring style consistently
- Explain *why*, not *what*; code should be self-explanatory
- Keep comments up to date with code changes

```python
def calculate_discount(price: float, percentage: float) -> float:
    """Calculate the discounted price.

    Args:
        price: Original price in dollars.
        percentage: Discount percentage (0-100).

    Returns:
        The price after applying the discount.

    Raises:
        ValueError: If percentage is not between 0 and 100.

    Examples:
        >>> calculate_discount(100.0, 20)
        80.0
    """
    if not 0 <= percentage <= 100:
        raise ValueError(f"percentage must be 0-100, got {percentage}")
    return price * (1 - percentage / 100)
```

---

## Pre-Review Checklist

Before submitting code for review, verify:

- [ ] `uv run ruff format .` applied
- [ ] `uv run ruff check .` passes with no errors
- [ ] `uv run mypy src/` passes with no errors
- [ ] All public functions have type hints
- [ ] All public functions have docstrings
- [ ] No bare `except:` clauses
- [ ] No mutable default arguments
- [ ] Tests cover main functionality
- [ ] No hardcoded secrets or credentials
