---
source_course: "go-architecture"
source_lesson: "go-architecture-package-design"
---

# Designing Good Packages

## Naming Conventions

*   Short, concise, lowercase names.
*   Avoid stuttering: `http.HTTPServer` → `http.Server`.
*   Package name becomes the prefix for exported identifiers.

## Package Cohesion

*   Each package should do one thing well.
*   Related functionality together.
*   Avoid "grab bag" packages.

```go
// Bad: Generic utils
package utils
func FormatDate()
func ParseJSON()
func HashPassword()

// Good: Focused packages
package dates
package jsonutil  // If needed
package auth
```

## Dependency Direction

*   Higher-level packages import lower-level.
*   Never import `cmd/` from library code.
*   Use interfaces to break dependencies.

## Exporting

*   Export only what's needed.
*   Start with everything unexported.
*   Export when there's a clear need.

## Code Examples

**Well-Designed Package**

```go
// Package auth provides authentication and authorization.
package auth

// User represents an authenticated user.
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


---

> 📘 *This lesson is part of the [Go Architecture & Design](https://stanza.dev/courses/go-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*