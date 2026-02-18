---
source_course: "go-architecture"
source_lesson: "go-architecture-mocking-strategies"
---

# Mocking with Interfaces

To write unit tests, we often need to mock external dependencies (DB, API). In Go, we do this by accepting interfaces.

## The Pattern
1.  Define an interface for the dependency.
2.  Implement the real version.
3.  Implement a mock version for tests.

```go
type Database interface {
    GetUser(id int) (*User, error)
}

type RealDB struct { /* sql.DB */ }
func (db *RealDB) GetUser(id int) (*User, error) { /* ... */ }

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

## Consumer-Defined Interfaces

Define interfaces where you use them, not where you implement them:

```go
// In your service package
type userGetter interface {
    GetUser(id int) (*User, error)
}
```

## Code Examples

**Using Mocks**

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


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*