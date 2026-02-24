---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-values"
---

# Context Values

## Introduction
`context.WithValue` allows attaching request-scoped data to a context — things like user IDs, trace IDs, and authentication tokens. However, it is frequently misused. Understanding its proper scope is critical.

## Key Concepts
- **context.WithValue:** Attaches a key-value pair to a context, creating a new derived context.
- **Custom Key Type:** Always use an unexported custom type for keys to prevent collisions between packages.
- **Transit Data Only:** Context values should carry data that transits through API boundaries, not replace function parameters.

## Real World Context
In a distributed system, a trace ID is generated when a request enters the system and must be propagated through every service call, database query, and log statement. Context values are the standard way to carry this trace ID without adding it to every function signature.

## Deep Dive
`context.WithValue` allows passing request-scoped data:

```go
ctx := context.WithValue(parent, key, "value")
val := ctx.Value(key)
```

### Best Practices for Keys
Use custom types for keys to avoid collisions with other packages:

```go
type contextKey string
const userIDKey contextKey = "userID"

func WithUserID(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, userIDKey, id)
}

func GetUserID(ctx context.Context) string {
    id, _ := ctx.Value(userIDKey).(string)
    return id
}
```

### What NOT to Put in Context Values
- Function parameters (use regular arguments instead).
- Optional configuration (use functional options).
- Mutable state (context values should be immutable).

## Common Pitfalls
1. **Using string keys** — String keys like `"user"` are global and easily collide with other packages using the same key. Always use an unexported custom type.
2. **Using context values for optional parameters** — This hides dependencies and makes code harder to test. Use explicit function parameters.

## Best Practices
1. **Provide getter/setter functions** — Wrap `WithValue` and `Value` in typed helper functions to ensure type safety.
2. **Limit to transit data** — Only use context values for cross-cutting concerns like trace IDs, request IDs, and auth tokens.

## Summary
- `context.WithValue` attaches key-value pairs for request-scoped transit data.
- Always use unexported custom types for keys to prevent collisions.
- Do not use context values as optional function parameters.
- Provide typed getter/setter helpers for type safety.

## Code Examples

**Type-safe context values using an unexported key type — prevents collisions and provides a clean API**

```go
type key int
const userKey key = 0

func WithUser(ctx context.Context, u *User) context.Context {
    return context.WithValue(ctx, userKey, u)
}

func GetUser(ctx context.Context) *User {
    u, _ := ctx.Value(userKey).(*User)
    return u
}
```


## Resources

- [Go Blog: Contexts and structs](https://go.dev/blog/context-and-structs) — Official guidance on when to use context values vs struct fields

---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*