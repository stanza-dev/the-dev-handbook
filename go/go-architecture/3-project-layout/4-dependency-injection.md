---
source_course: "go-architecture"
source_lesson: "go-architecture-dependency-injection"
---

# Dependency Injection

## Introduction
Dependency injection (DI) is the practice of passing dependencies into a function or struct rather than creating them internally. Go favors explicit DI through constructors—no magic frameworks needed. This pattern is the foundation of testable, modular Go applications.

## Key Concepts
- **Constructor Injection**: Passing dependencies as parameters to a `New*` function that returns a struct.
- **Wiring in main()**: The `main` function creates all real dependencies and injects them into services.
- **No Global State**: Dependencies are struct fields, not package-level variables.

## Real World Context
Your `UserService` needs a database, a cache, and a logger. If it creates these internally, you cannot test it without a running database. Constructor injection lets you pass mocks in tests and real implementations in production.

## Deep Dive
The constructor pattern accepts interfaces and returns a concrete struct:

```go
type UserService struct {
    db     Database
    cache  Cache
    logger Logger
}

func NewUserService(db Database, cache Cache, logger Logger) *UserService {
    return &UserService{
        db:     db,
        cache:  cache,
        logger: logger,
    }
}
```

All wiring happens in `main()`:

```go
func main() {
    // Create real dependencies
    db := database.New(os.Getenv("DB_URL"))
    cache := redis.New(os.Getenv("REDIS_URL"))
    logger := slog.Default()

    // Inject into services
    userSvc := service.NewUserService(db, cache, logger)
    orderSvc := service.NewOrderService(db, logger)

    // Create server with services
    srv := server.New(userSvc, orderSvc)
    srv.Run()
}
```

This approach makes the dependency graph explicit. You can trace every dependency by reading `main()`. There is no hidden state, no service locator, and no runtime reflection.

## Common Pitfalls
1. **Using package-level variables for dependencies** — Global state makes testing impossible and creates hidden coupling. Always inject through constructors.
2. **Using a DI framework** — Go's simplicity means you rarely need frameworks like Wire or Dig. Manual DI in main() is usually clearer.

## Best Practices
1. **Accept interfaces, store as struct fields** — This enables mocking in tests and swapping implementations.
2. **Keep main() as the composition root** — All dependency creation and wiring should happen in main or a dedicated setup function.

## Summary
- Pass dependencies through constructors, not global state.
- Wire everything in main() for explicit dependency graphs.
- Accept interfaces in constructors for testability.
- Manual DI is usually clearer than framework-based DI in Go.

## Code Examples

**An Application struct that serves as the composition root, wiring all dependencies together in a single place for clarity**

```go
// Application struct wires all dependencies together.
// This is the composition root of the application.
type Application struct {
    config  *Config
    db      *sql.DB
    cache   *redis.Client
    server  *http.Server
}

func NewApplication(cfg *Config) (*Application, error) {
    db, err := sql.Open("postgres", cfg.DatabaseURL)
    if err != nil {
        return nil, fmt.Errorf("opening database: %w", err)
    }
    return &Application{config: cfg, db: db}, nil
}
```


## Resources

- [Go Wiki — Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) — Official Go wiki with code review guidance on dependency management

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*