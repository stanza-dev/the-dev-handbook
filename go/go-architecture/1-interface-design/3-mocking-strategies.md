---
source_course: "go-architecture"
source_lesson: "go-architecture-mocking-strategies"
---

# Mocking Strategies

## Introduction
Unit testing code with external dependencies—databases, APIs, file systems—requires replacing those dependencies with controlled substitutes. Go's implicit interfaces make mocking straightforward without any framework or code generation. Mastering this pattern is essential for writing testable Go applications.

## Key Concepts
- **Mock**: A test double that replaces a real dependency with a controlled implementation.
- **Consumer-Defined Interface**: Defining the interface where it is used (not where it is implemented), specifying only the methods you need.
- **Dependency Injection**: Passing dependencies into a function or struct rather than creating them internally.

## Real World Context
You are building a user service that fetches data from PostgreSQL. In production, you use `*sql.DB`. In tests, you need a fake that returns predefined data and errors. By accepting a `Database` interface, you can swap implementations without changing your service code.

## Deep Dive
The pattern has three steps. First, define an interface for the dependency:

```go
type Database interface {
    GetUser(id int) (*User, error)
}
```

Second, implement the real version:

```go
type RealDB struct { pool *sql.DB }
func (db *RealDB) GetUser(id int) (*User, error) {
    // Real database query
}
```

Third, implement a mock for tests:

```go
type MockDB struct {
    Users map[int]*User
    Err   error
}
func (m *MockDB) GetUser(id int) (*User, error) {
    if m.Err != nil {
        return nil, m.Err
    }
    return m.Users[id], nil
}
```

A key Go idiom is consumer-defined interfaces. Define the interface in the package that uses it, not the one that implements it:

```go
// In your service package, not the database package
type userGetter interface {
    GetUser(id int) (*User, error)
}
```

This way, the service only depends on the methods it actually calls.

## Common Pitfalls
1. **Defining interfaces in the producer package** — This couples the consumer to the producer's full API. Define interfaces where you use them.
2. **Making mock interfaces too large** — If your mock has 15 methods, your interface is too big. Break it into smaller interfaces.

## Best Practices
1. **Keep mock interfaces minimal** — Only include the methods the consumer actually calls. One or two methods is ideal.
2. **Test error paths** — Use your mock's `Err` field to simulate failures and verify your service handles them correctly.

## Summary
- Define interfaces where you use them (consumer-defined), not where you implement them.
- Mocking in Go requires no frameworks—just implement the interface.
- Keep mock interfaces small for maximum flexibility.
- Always test both success and error paths with your mocks.

## Code Examples

**A complete test using a mock database — the mock satisfies the Database interface implicitly, and we control exactly what data it returns**

```go
func TestService(t *testing.T) {
    mock := &MockDB{Users: map[int]*User{
        1: {Name: "Test"},
    }}
    svc := NewService(mock)

    user, err := svc.GetUser(1)
    if err != nil {
        t.Fatal(err)
    }
    if user.Name != "Test" {
        t.Errorf("want Test, got %s", user.Name)
    }
}
```


## Resources

- [Go Wiki — Table-Driven Tests](https://github.com/golang/go/wiki/TableDrivenTests) — Official Go wiki guide on writing effective table-driven tests
- [Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces) — Official guide on how Go interfaces work and their implicit satisfaction model

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*