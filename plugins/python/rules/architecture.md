# Architecture

Follow [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) and [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) principles adapted for Python's dynamic nature and protocol system.

## SOLID Principles

### Single Responsibility Principle (SRP)

- Each module should do one thing well
- Keep classes focused; avoid "god" classes handling multiple concerns
- Separate business logic from infrastructure (logging, HTTP, database)
- If a class needs multiple reasons to change, split it

```python
# Bad: mixing concerns
class UserService:
    def __init__(self, db: Database, logger: Logger):
        self.db = db
        self.logger = logger

    def create(self, user: User) -> None:
        self.logger.info(f"creating user {user.name}")  # logging concern
        # validation, business logic, and database all mixed

# Good: separated concerns
class UserService:
    def __init__(self, repo: UserRepository):
        self.repo = repo

    def create(self, user: User) -> None:
        # only business logic here
        self.repo.save(user)
```

### Open/Closed Principle (OCP)

- Design for extension without modification
- Use protocols/abstract base classes to allow new implementations
- Leverage composition over inheritance
- Add new behavior by implementing existing protocols

```python
from typing import Protocol

# Open for extension via protocol
class Notifier(Protocol):
    def notify(self, message: str) -> None: ...

# Closed for modification - add new notifiers without changing existing code
class EmailNotifier:
    def notify(self, message: str) -> None: ...

class SlackNotifier:
    def notify(self, message: str) -> None: ...

class SmsNotifier:
    def notify(self, message: str) -> None: ...
```

### Liskov Substitution Principle (LSP)

- Implementations must honor the contract defined by protocols
- Subtypes must be substitutable for their base types
- Don't violate expected behavior in implementations

```python
from typing import Protocol

# Protocol defines the contract
class Reader(Protocol):
    def read(self, size: int = -1) -> bytes:
        """Read bytes. Returns empty bytes at EOF."""
        ...

# All implementations must honor: return empty bytes at EOF
```

### Interface Segregation Principle (ISP)

- Keep protocols small and focused (1-3 methods ideal)
- Clients should not depend on methods they don't use
- Define protocols at the consumer, not the provider
- Compose larger protocols from smaller ones when needed

```python
from typing import Protocol

# Bad: fat protocol
class UserStore(Protocol):
    def create(self, user: User) -> None: ...
    def get(self, id: str) -> User: ...
    def update(self, user: User) -> None: ...
    def delete(self, id: str) -> None: ...
    def list_all(self) -> list[User]: ...
    def search(self, query: str) -> list[User]: ...
    def export(self) -> bytes: ...

# Good: segregated protocols
class UserReader(Protocol):
    def get(self, id: str) -> User: ...

class UserWriter(Protocol):
    def create(self, user: User) -> None: ...
    def update(self, user: User) -> None: ...

class UserDeleter(Protocol):
    def delete(self, id: str) -> None: ...
```

### Dependency Inversion Principle (DIP)

- High-level modules should not depend on low-level modules
- Both should depend on abstractions (protocols)
- Accept protocols, return concrete types
- Inject dependencies; don't create them internally

```python
from typing import Protocol

# Bad: direct dependency on concrete type
class OrderService:
    def __init__(self):
        self.db = PostgresDatabase()  # creates its own dependency

# Good: depend on abstraction
class OrderRepository(Protocol):
    def save(self, order: Order) -> None: ...
    def find_by_id(self, id: str) -> Order | None: ...

class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo  # dependency injected
```

## Layered Architecture

Organize code into distinct layers with clear responsibilities:

| Layer | Responsibility | Dependencies |
|-------|---------------|--------------|
| Domain | Business entities, rules, protocols | None |
| Application | Use cases, orchestration | Domain |
| Adapters | External system integration | Application, Domain |
| Infrastructure | Frameworks, drivers, tools | All layers |

### Dependency Rule

Dependencies point inward only:
- Domain layer has no external dependencies
- Application layer depends only on Domain
- Adapters depend on Application and Domain
- Infrastructure can depend on all layers

```
┌─────────────────────────────────────────┐
│            Infrastructure               │
│  ┌─────────────────────────────────┐   │
│  │           Adapters              │   │
│  │  ┌─────────────────────────┐   │   │
│  │  │      Application        │   │   │
│  │  │  ┌─────────────────┐   │   │   │
│  │  │  │     Domain      │   │   │   │
│  │  │  └─────────────────┘   │   │   │
│  │  └─────────────────────────┘   │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

## Recommended Package Structure

```
src/mypackage/
├── __init__.py
├── domain/               # Business entities and protocols
│   ├── __init__.py
│   ├── user.py           # User entity
│   ├── order.py          # Order entity
│   └── repository.py     # Repository protocols
├── application/          # Use cases / services
│   ├── __init__.py
│   ├── user_service.py
│   └── order_service.py
├── adapters/
│   ├── __init__.py
│   ├── api/              # FastAPI/Flask handlers (inbound)
│   ├── cli/              # CLI commands (inbound)
│   └── postgres/         # Database implementation (outbound)
└── infrastructure/
    ├── __init__.py
    ├── config.py         # Configuration loading
    └── container.py      # Dependency injection
```

## Ports and Adapters

### Inbound Ports (Driving)

Entry points that invoke domain logic:
- HTTP/REST handlers (FastAPI, Flask)
- CLI commands (Click, Typer)
- Message queue consumers
- GraphQL resolvers

### Outbound Ports (Driven)

Protocols the domain needs:
- Repository protocols
- External API clients
- Message publishers
- Cache protocols

```python
from typing import Protocol
from dataclasses import dataclass

# Outbound port (defined in domain)
class UserRepository(Protocol):
    async def save(self, user: User) -> None: ...
    async def find_by_id(self, id: str) -> User | None: ...

# Adapter implements the port
@dataclass
class PostgresUserRepository:
    pool: asyncpg.Pool

    async def save(self, user: User) -> None:
        # PostgreSQL-specific implementation
        await self.pool.execute(
            "INSERT INTO users (id, name, email) VALUES ($1, $2, $3)",
            user.id, user.name, user.email,
        )

    async def find_by_id(self, id: str) -> User | None:
        row = await self.pool.fetchrow(
            "SELECT * FROM users WHERE id = $1", id
        )
        return User(**row) if row else None
```

## Dependency Injection

```python
# infrastructure/container.py
from dataclasses import dataclass
from mypackage.domain.repository import UserRepository
from mypackage.application.user_service import UserService
from mypackage.adapters.postgres import PostgresUserRepository

@dataclass
class Container:
    """Dependency injection container."""
    user_repo: UserRepository
    user_service: UserService

    @classmethod
    async def create(cls, config: Config) -> "Container":
        pool = await asyncpg.create_pool(config.database_url)
        user_repo = PostgresUserRepository(pool)
        user_service = UserService(user_repo)
        return cls(user_repo=user_repo, user_service=user_service)
```

## Best Practices

- Keep the domain layer pure; no framework dependencies
- Use dependency injection for all external dependencies
- Define protocols in the module that uses them
- Test domain logic without infrastructure
- Use dataclasses or Pydantic for value objects
- Avoid circular imports between modules

## Anti-Patterns to Avoid

- **Anemic Domain Model**: Domain objects with only data, no behavior
- **Big Ball of Mud**: No clear boundaries between concerns
- **God Module**: One module that does everything
- **Leaky Abstractions**: Infrastructure details bleeding into domain
- **Circular Imports**: Modules that import each other

---

## Architecture Checklist

Before finalizing design decisions, verify:

- [ ] Domain layer has no external dependencies
- [ ] Dependencies point inward (toward domain)
- [ ] Protocols defined at consumer, not provider
- [ ] Each module has a single, clear responsibility
- [ ] External systems accessed through protocols
- [ ] Business logic testable without infrastructure
- [ ] No circular imports between modules
- [ ] Dependency injection used for external dependencies
