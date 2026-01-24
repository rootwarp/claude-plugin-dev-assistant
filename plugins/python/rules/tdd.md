# Test-Driven Development (TDD)

- Write tests BEFORE implementing functionality
- Follow the Red-Green-Refactor cycle:
  1. Write a failing test that defines expected behavior
  2. Write minimal code to make the test pass
  3. Refactor while keeping tests green
- Each test should test ONE behavior
- Use descriptive test names: `test_function_scenario_expected_behavior`
- Place tests in `tests/` directory mirroring source structure
- Use `pytest` as the test framework
- Mock external dependencies using `unittest.mock` or `pytest-mock`
- Aim for meaningful coverage, not 100% coverage
- Run tests before committing: `pytest`

## Test Organization

```python
# tests/test_user.py
import pytest
from mypackage.user import User, ValidationError

class TestUserCreation:
    """Tests for User creation."""

    def test_new_user_valid_input_creates_user(self):
        user = User(name="Alice", email="alice@example.com")
        assert user.name == "Alice"
        assert user.email == "alice@example.com"

    def test_new_user_empty_name_raises_error(self):
        with pytest.raises(ValidationError, match="name cannot be empty"):
            User(name="", email="alice@example.com")

    def test_new_user_invalid_email_raises_error(self):
        with pytest.raises(ValidationError, match="invalid email"):
            User(name="Alice", email="not-an-email")
```

## Fixtures

Use `conftest.py` for shared fixtures:

```python
# tests/conftest.py
import pytest
from mypackage.config import Config
from mypackage.database import Database

@pytest.fixture
def config() -> Config:
    """Provide test configuration."""
    return Config(
        database_url="sqlite:///:memory:",
        debug=True,
    )

@pytest.fixture
def db(config: Config) -> Database:
    """Provide test database."""
    database = Database(config.database_url)
    database.create_tables()
    yield database
    database.drop_tables()
```

## Parametrized Tests

```python
import pytest

@pytest.mark.parametrize(
    "input_value,expected",
    [
        (2, 4),
        (0, 0),
        (-3, 9),
        (1.5, 2.25),
    ],
)
def test_square(input_value: float, expected: float):
    assert square(input_value) == expected

@pytest.mark.parametrize(
    "email,is_valid",
    [
        ("user@example.com", True),
        ("user@sub.example.com", True),
        ("invalid", False),
        ("@example.com", False),
        ("user@", False),
    ],
)
def test_email_validation(email: str, is_valid: bool):
    assert validate_email(email) == is_valid
```

## Mocking

```python
from unittest.mock import Mock, patch, AsyncMock
import pytest

class TestEmailService:
    def test_send_email_calls_smtp_client(self):
        # Arrange
        mock_smtp = Mock()
        service = EmailService(smtp_client=mock_smtp)

        # Act
        service.send("to@example.com", "Subject", "Body")

        # Assert
        mock_smtp.send.assert_called_once_with(
            to="to@example.com",
            subject="Subject",
            body="Body",
        )

    @patch("mypackage.email.smtp_connect")
    def test_send_email_with_patch(self, mock_connect):
        mock_connect.return_value = Mock()
        service = EmailService()
        service.send("to@example.com", "Subject", "Body")
        mock_connect.assert_called_once()

class TestAsyncService:
    @pytest.mark.asyncio
    async def test_fetch_data(self):
        mock_client = AsyncMock()
        mock_client.get.return_value = {"data": "value"}

        service = DataService(client=mock_client)
        result = await service.fetch("key")

        assert result == {"data": "value"}
        mock_client.get.assert_awaited_once_with("key")
```

## Async Tests

```python
import pytest

@pytest.mark.asyncio
async def test_async_fetch():
    client = AsyncClient()
    result = await client.fetch("https://api.example.com/data")
    assert result.status == 200
```

## Test Coverage

```bash
# Run with coverage
pytest --cov=mypackage --cov-report=html

# Fail if coverage below threshold
pytest --cov=mypackage --cov-fail-under=80
```

## pytest.ini / pyproject.toml Configuration

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --strict-markers"
markers = [
    "slow: marks tests as slow",
    "integration: marks tests as integration tests",
]
asyncio_mode = "auto"
```
