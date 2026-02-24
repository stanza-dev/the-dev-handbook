---
source_course: "go-architecture"
source_lesson: "go-architecture-package-design"
---

# Package Design Principles

## Introduction
Packages are Go's primary unit of organization, encapsulation, and reuse. A well-designed package has a clear name, a focused purpose, and minimal exported surface. Poor package design leads to tight coupling and confusing import graphs. These principles will guide you toward clean, maintainable Go code.

## Key Concepts
- **Package Naming**: Short, lowercase, no underscores. The package name becomes the prefix for all exported identifiers.
- **Package Cohesion**: Each package should do one thing well. Related functionality goes together.
- **Export Minimalism**: Start with everything unexported. Only export when there is a clear external need.

## Real World Context
You create a `utils` package with `FormatDate()`, `ParseJSON()`, and `HashPassword()`. Six months later it has 50 unrelated functions and every package imports it, creating a dependency bottleneck. Focused packages like `dates`, `auth`, and `jsonutil` avoid this trap.

## Deep Dive
Go package naming avoids stuttering. Since the package name is the prefix, `http.HTTPServer` becomes `http.Server`:

```go
// Bad: Generic utils
package utils
func FormatDate() {}
func ParseJSON() {}
func HashPassword() {}

// Good: Focused packages
package dates    // dates.Format()
package auth     // auth.HashPassword()
```

Dependency direction should flow from high-level to low-level. The `/cmd` layer imports `/internal`, which imports shared libraries. Never import `/cmd` from library code.

Use interfaces to break dependencies when two packages need to communicate:

```go
// Package auth provides authentication and authorization.
package auth

type User struct {
    ID    string
    Email string
    role  string  // Unexported: internal detail
}

// Authenticate validates credentials and returns a User.
func Authenticate(email, password string) (*User, error) {
    // ...
}
```

Notice that `role` is unexported. Only export what external consumers need.

## Common Pitfalls
1. **Creating `utils` or `helpers` packages** — These inevitably become grab bags. Name packages by their domain.
2. **Over-exporting** — Exporting everything makes refactoring painful. Start unexported and promote to exported only when needed.

## Best Practices
1. **Name packages by what they provide, not what they contain** — `auth` provides authentication, `email` provides email sending.
2. **Keep dependency graphs acyclic** — Use interfaces at package boundaries to invert dependencies when needed.

## Summary
- Package names are short, lowercase, and describe what the package provides.
- Avoid stuttering: `http.Server`, not `http.HTTPServer`.
- Each package should have a single, focused responsibility.
- Export only what is needed; default to unexported.

## Code Examples

**A well-designed auth package with clear naming, focused responsibility, and minimal exports — the role field is intentionally unexported**

```go
// Package auth provides authentication and authorization.
package auth

// User represents an authenticated user.
type User struct {
    ID    string
    Email string
    role  string  // Unexported: internal implementation detail
}

// Authenticate validates credentials and returns a User.
func Authenticate(email, password string) (*User, error) {
    // Implementation...
    return &User{ID: "123", Email: email, role: "user"}, nil
}
```


## Resources

- [Effective Go — Package names](https://go.dev/doc/effective_go#names) — Official naming conventions for Go packages and identifiers
- [Go Blog — Package names](https://go.dev/blog/package-names) — Detailed guidance on choosing good Go package names

---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*