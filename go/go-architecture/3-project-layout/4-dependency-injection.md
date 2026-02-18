---
source_course: "go-architecture"
source_lesson: "go-architecture-dependency-injection"
---

# Dependency Injection in Go

Go favors explicit dependency injection through constructors.

## Constructor Injection

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

## Wire Up in main()

```go
func main() {
    // Create dependencies
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

## Benefits

*   Easy to test (inject mocks).
*   Explicit dependencies.
*   No global state.

## Code Examples

**Application Wiring**

```go
// Application struct wires everything together
type Application struct {
    config  *Config
    db      *sql.DB
    cache   *redis.Client
    server  *http.Server
}

func NewApplication(cfg *Config) (*Application, error) {
    db, err := sql.Open("postgres", cfg.DatabaseURL)
    if err != nil {
        return nil, err
    }
    // ... wire up other dependencies
    return &Application{config: cfg, db: db}, nil
}
```


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*