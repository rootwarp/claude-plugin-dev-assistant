# Architecture

Follow [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) and [Hexagonal Architecture](https://alistairhttps://alistair.cockburn.us/hexagonal-architecture/) principles adapted for Go's idiomatic style.

## SOLID Principles

### Single Responsibility Principle (SRP)

- Each package should do one thing well
- Keep structs focused; avoid "god" structs handling multiple concerns
- Separate business logic from infrastructure (logging, HTTP, database)
- If a struct needs multiple reasons to change, split it

```go
// Bad: mixing concerns
type UserService struct {
    db     *sql.DB
    logger *log.Logger
}

func (s *UserService) Create(u User) error {
    s.logger.Printf("creating user %s", u.Name)  // logging concern
    // validation, business logic, and database all mixed
}

// Good: separated concerns
type UserService struct {
    repo UserRepository
}

func (s *UserService) Create(u User) error {
    // only business logic here
}
```

### Open/Closed Principle (OCP)

- Design for extension without modification
- Use interfaces to allow new implementations
- Leverage composition over inheritance
- Add new behavior by implementing existing interfaces

```go
// Open for extension via interface
type Notifier interface {
    Notify(message string) error
}

// Closed for modification - add new notifiers without changing existing code
type EmailNotifier struct{}
type SlackNotifier struct{}
type SMSNotifier struct{}
```

### Liskov Substitution Principle (LSP)

- Implementations must honor the contract defined by interfaces
- Subtypes must be substitutable for their base types
- Don't violate expected behavior in implementations

```go
// Interface defines the contract
type Reader interface {
    Read(p []byte) (n int, err error)
}

// All implementations must honor: return io.EOF at end, n <= len(p)
```

### Interface Segregation Principle (ISP)

- Keep interfaces small and focused (1-3 methods ideal)
- Clients should not depend on methods they don't use
- Define interfaces at the consumer, not the provider
- Compose larger interfaces from smaller ones when needed

```go
// Bad: fat interface
type UserStore interface {
    Create(User) error
    Get(id string) (User, error)
    Update(User) error
    Delete(id string) error
    List() ([]User, error)
    Search(query string) ([]User, error)
    Export() ([]byte, error)
}

// Good: segregated interfaces
type UserReader interface {
    Get(id string) (User, error)
}

type UserWriter interface {
    Create(User) error
    Update(User) error
}

type UserDeleter interface {
    Delete(id string) error
}
```

### Dependency Inversion Principle (DIP)

- High-level modules should not depend on low-level modules
- Both should depend on abstractions (interfaces)
- Accept interfaces, return concrete types
- Inject dependencies; don't create them internally

```go
// Bad: direct dependency on concrete type
type OrderService struct {
    db *sql.DB  // depends on concrete database
}

// Good: depend on abstraction
type OrderService struct {
    repo OrderRepository  // depends on interface
}

func NewOrderService(repo OrderRepository) *OrderService {
    return &OrderService{repo: repo}
}
```

## Layered Architecture

Organize code into distinct layers with clear responsibilities:

| Layer | Responsibility | Dependencies |
|-------|---------------|--------------|
| Domain | Business entities, rules, interfaces | None |
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
internal/
├── domain/           # Business entities and interfaces
│   ├── user.go       # User entity
│   ├── order.go      # Order entity
│   └── repository.go # Repository interfaces
├── application/      # Use cases / services
│   ├── user_service.go
│   └── order_service.go
├── adapters/
│   ├── http/         # HTTP handlers (inbound)
│   ├── grpc/         # gRPC handlers (inbound)
│   └── postgres/     # Database implementation (outbound)
└── infrastructure/
    ├── config/       # Configuration loading
    └── wire/         # Dependency injection
```

## Ports and Adapters

### Inbound Ports (Driving)

Entry points that invoke domain logic:
- HTTP/REST handlers
- gRPC services
- CLI commands
- Message queue consumers

### Outbound Ports (Driven)

Interfaces the domain needs:
- Repository interfaces
- External API clients
- Message publishers
- Cache interfaces

```go
// Outbound port (defined in domain)
type UserRepository interface {
    Save(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id string) (*User, error)
}

// Adapter implements the port
type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) Save(ctx context.Context, user *User) error {
    // PostgreSQL-specific implementation
}
```

## Best Practices

- Keep the domain layer pure; no framework dependencies
- Use dependency injection for all external dependencies
- Define interfaces in the package that uses them
- Test domain logic without infrastructure
- Use constructor functions to enforce required dependencies
- Avoid circular dependencies between packages

## Anti-Patterns to Avoid

- **Anemic Domain Model**: Domain objects with only getters/setters, no behavior
- **Big Ball of Mud**: No clear boundaries between concerns
- **God Package**: One package that does everything
- **Leaky Abstractions**: Infrastructure details bleeding into domain
- **Circular Dependencies**: Packages that import each other

---

## Architecture Checklist

Before finalizing design decisions, verify:

- [ ] Domain layer has no external dependencies
- [ ] Dependencies point inward (toward domain)
- [ ] Interfaces defined at consumer, not provider
- [ ] Each package has a single, clear responsibility
- [ ] External systems accessed through interfaces
- [ ] Business logic testable without infrastructure
- [ ] No circular dependencies between packages
- [ ] Constructor functions enforce required dependencies
