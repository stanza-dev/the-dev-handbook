---
source_course: "go-concurrency"
source_lesson: "go-concurrency-context-values"
---

# Context Values

`context.WithValue` allows passing request-scoped data like User IDs, Trace IDs, or Auth tokens.

```go
ctx := context.WithValue(parent, key, "value")
val := ctx.Value(key)
```

## Best Practices

*   **Use custom types for keys** to avoid collisions with other packages.
*   **Do not use it for optional function parameters.**
*   Use it only for transit data that passes through layers.

```go
// Define key type in your package
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

## Code Examples

**Type-Safe Context Values**

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


---

> 📘 *This lesson is part of the [Go Concurrency Patterns](https://stanza.dev/courses/go-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*