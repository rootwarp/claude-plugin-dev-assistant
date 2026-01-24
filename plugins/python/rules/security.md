# Security

- Never hardcode secrets, credentials, or API keys
- Use environment variables or secret managers for configuration
- Validate and sanitize ALL user input
- Use parameterized queries to prevent SQL injection (SQLAlchemy, asyncpg)
- Escape output properly to prevent XSS in web applications
- Use `secrets` module for cryptographic randomness, not `random`
- Set appropriate timeouts on HTTP clients and network operations
- Use TLS/HTTPS for all network communication
- Run `bandit` to catch common security issues
- Keep dependencies updated; audit with `pip-audit` or `safety`
- Follow principle of least privilege for file permissions
- Log security events but never log sensitive data
- Use `python-dotenv` for local development only, not production

## Input Validation

```python
from pydantic import BaseModel, EmailStr, Field, field_validator

class UserInput(BaseModel):
    """Validated user input."""
    name: str = Field(min_length=1, max_length=100)
    email: EmailStr
    age: int = Field(ge=0, le=150)

    @field_validator("name")
    @classmethod
    def name_must_not_contain_script(cls, v: str) -> str:
        if "<script" in v.lower():
            raise ValueError("Invalid characters in name")
        return v.strip()
```

## SQL Injection Prevention

```python
# Bad: string formatting
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# Good: parameterized query
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# Good: SQLAlchemy
from sqlalchemy import select
stmt = select(User).where(User.id == user_id)
result = session.execute(stmt)
```

## Secrets Management

```python
import os
from functools import lru_cache
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """Application settings from environment."""
    database_url: str
    api_key: str
    secret_key: str

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}

@lru_cache
def get_settings() -> Settings:
    return Settings()

# Usage
settings = get_settings()
```

## Cryptographic Randomness

```python
import secrets

# Good: cryptographically secure
token = secrets.token_urlsafe(32)
api_key = secrets.token_hex(32)

# Bad: not secure for secrets
import random
token = ''.join(random.choices('abc123', k=32))  # Predictable!
```

## Password Hashing

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["argon2", "bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

## HTTP Client Security

```python
import httpx

# Set timeouts to prevent hanging
client = httpx.Client(
    timeout=httpx.Timeout(10.0, connect=5.0),
    follow_redirects=True,
    max_redirects=5,
)

# Verify SSL certificates (default, but be explicit)
async with httpx.AsyncClient(verify=True) as client:
    response = await client.get("https://api.example.com")
```

## Security Scanning

```bash
# Install security tools
pip install bandit pip-audit safety

# Static analysis for security issues
bandit -r src/

# Check for vulnerable dependencies
pip-audit
safety check

# In CI/CD
bandit -r src/ -f json -o bandit-report.json
pip-audit --format=json > audit-report.json
```

## Common Vulnerabilities to Avoid

| Vulnerability | Prevention |
|--------------|------------|
| SQL Injection | Parameterized queries |
| XSS | Escape output, use templating engines |
| CSRF | Use CSRF tokens |
| Path Traversal | Validate file paths, use `pathlib` |
| Command Injection | Avoid `shell=True`, use `shlex` |
| Deserialization | Don't unpickle untrusted data |
| Hardcoded Secrets | Environment variables, secret managers |

```python
# Bad: command injection risk
import subprocess
subprocess.run(f"ls {user_input}", shell=True)  # Dangerous!

# Good: no shell, list arguments
import shlex
subprocess.run(["ls", user_input], shell=False)
```
