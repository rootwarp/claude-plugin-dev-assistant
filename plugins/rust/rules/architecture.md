# Architecture

Follow [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) and [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) principles adapted for Rust's ownership model and trait system.

## SOLID Principles

### Single Responsibility Principle (SRP)

- Each module should do one thing well
- Keep structs focused; avoid "god" structs handling multiple concerns
- Separate business logic from infrastructure (logging, HTTP, database)
- If a struct needs multiple reasons to change, split it

```rust
// Bad: mixing concerns
struct UserService {
    db: Pool<Postgres>,
    logger: Logger,
}

impl UserService {
    fn create(&self, user: User) -> Result<(), Error> {
        self.logger.info(&format!("creating user {}", user.name));
        // validation, business logic, and database all mixed
    }
}

// Good: separated concerns
struct UserService<R: UserRepository> {
    repo: R,
}

impl<R: UserRepository> UserService<R> {
    fn create(&self, user: User) -> Result<(), Error> {
        // only business logic here
        self.repo.save(user)
    }
}
```

### Open/Closed Principle (OCP)

- Design for extension without modification
- Use traits to allow new implementations
- Leverage composition over inheritance
- Add new behavior by implementing existing traits

```rust
// Open for extension via trait
trait Notifier: Send + Sync {
    fn notify(&self, message: &str) -> Result<(), Error>;
}

// Closed for modification - add new notifiers without changing existing code
struct EmailNotifier { /* ... */ }
struct SlackNotifier { /* ... */ }
struct SmsNotifier { /* ... */ }

impl Notifier for EmailNotifier { /* ... */ }
impl Notifier for SlackNotifier { /* ... */ }
impl Notifier for SmsNotifier { /* ... */ }
```

### Liskov Substitution Principle (LSP)

- Implementations must honor the contract defined by traits
- Subtypes must be substitutable for their base types
- Don't violate expected behavior in implementations

```rust
// Trait defines the contract
trait Reader {
    /// Reads bytes into buffer, returns bytes read.
    /// Returns 0 at EOF, never negative.
    fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>;
}

// All implementations must honor: return 0 at EOF, n <= buf.len()
```

### Interface Segregation Principle (ISP)

- Keep traits small and focused (1-3 methods ideal)
- Clients should not depend on methods they don't use
- Define traits at the consumer, not the provider
- Compose larger traits from smaller ones when needed

```rust
// Bad: fat trait
trait UserStore {
    fn create(&self, user: User) -> Result<(), Error>;
    fn get(&self, id: &str) -> Result<User, Error>;
    fn update(&self, user: User) -> Result<(), Error>;
    fn delete(&self, id: &str) -> Result<(), Error>;
    fn list(&self) -> Result<Vec<User>, Error>;
    fn search(&self, query: &str) -> Result<Vec<User>, Error>;
    fn export(&self) -> Result<Vec<u8>, Error>;
}

// Good: segregated traits
trait UserReader {
    fn get(&self, id: &str) -> Result<User, Error>;
}

trait UserWriter {
    fn create(&self, user: User) -> Result<(), Error>;
    fn update(&self, user: User) -> Result<(), Error>;
}

trait UserDeleter {
    fn delete(&self, id: &str) -> Result<(), Error>;
}

// Compose when needed
trait UserRepository: UserReader + UserWriter + UserDeleter {}
```

### Dependency Inversion Principle (DIP)

- High-level modules should not depend on low-level modules
- Both should depend on abstractions (traits)
- Accept traits, return concrete types
- Inject dependencies; don't create them internally

```rust
// Bad: direct dependency on concrete type
struct OrderService {
    db: Pool<Postgres>,  // depends on concrete database
}

// Good: depend on abstraction
struct OrderService<R: OrderRepository> {
    repo: R,
}

impl<R: OrderRepository> OrderService<R> {
    fn new(repo: R) -> Self {
        Self { repo }
    }
}

// Or use trait objects for runtime polymorphism
struct OrderService {
    repo: Box<dyn OrderRepository>,
}
```

## Layered Architecture

Organize code into distinct layers with clear responsibilities:

| Layer | Responsibility | Dependencies |
|-------|---------------|--------------|
| Domain | Business entities, rules, traits | None |
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

## Recommended Module Structure

```
src/
├── lib.rs            # Public API, re-exports
├── domain/           # Business entities and traits
│   ├── mod.rs
│   ├── user.rs       # User entity
│   ├── order.rs      # Order entity
│   └── repository.rs # Repository traits
├── application/      # Use cases / services
│   ├── mod.rs
│   ├── user_service.rs
│   └── order_service.rs
├── adapters/
│   ├── mod.rs
│   ├── http/         # HTTP handlers (inbound)
│   ├── grpc/         # gRPC handlers (inbound)
│   └── postgres/     # Database implementation (outbound)
└── infrastructure/
    ├── mod.rs
    ├── config.rs     # Configuration loading
    └── di.rs         # Dependency injection setup
```

## Ports and Adapters

### Inbound Ports (Driving)

Entry points that invoke domain logic:
- HTTP/REST handlers (axum, actix-web)
- gRPC services (tonic)
- CLI commands (clap)
- Message queue consumers

### Outbound Ports (Driven)

Traits the domain needs:
- Repository traits
- External API clients
- Message publishers
- Cache traits

```rust
// Outbound port (defined in domain)
#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn save(&self, user: &User) -> Result<(), Error>;
    async fn find_by_id(&self, id: &UserId) -> Result<Option<User>, Error>;
}

// Adapter implements the port
pub struct PostgresUserRepository {
    pool: Pool<Postgres>,
}

#[async_trait]
impl UserRepository for PostgresUserRepository {
    async fn save(&self, user: &User) -> Result<(), Error> {
        // PostgreSQL-specific implementation
    }
}
```

## Best Practices

- Keep the domain layer pure; no framework dependencies
- Use dependency injection via generics or trait objects
- Define traits in the module that uses them
- Test domain logic without infrastructure
- Use constructor functions to enforce required dependencies
- Avoid circular dependencies between modules

## Anti-Patterns to Avoid

- **Anemic Domain Model**: Domain objects with only getters/setters, no behavior
- **Big Ball of Mud**: No clear boundaries between concerns
- **God Module**: One module that does everything
- **Leaky Abstractions**: Infrastructure details bleeding into domain
- **Circular Dependencies**: Modules that import each other

---

## Architecture Checklist

Before finalizing design decisions, verify:

- [ ] Domain layer has no external dependencies
- [ ] Dependencies point inward (toward domain)
- [ ] Traits defined at consumer, not provider
- [ ] Each module has a single, clear responsibility
- [ ] External systems accessed through traits
- [ ] Business logic testable without infrastructure
- [ ] No circular dependencies between modules
- [ ] Constructor functions enforce required dependencies
