---
source_course: "go-web-services"
source_lesson: "go-web-services-routing-122"
---

# Enhanced Routing (Go 1.22+)

## Introduction
Go 1.22 upgraded the standard `http.ServeMux` to support HTTP method matching and path parameters, eliminating the need for third-party routers in many cases.

## Key Concepts
- **Method routing**: Prefixing a pattern with an HTTP method (e.g., `GET /users`) restricts the route to that method only.
- **Path parameters**: Named segments like `{id}` capture dynamic parts of the URL, accessed via `r.PathValue("id")`.
- **Wildcard segments**: `{path...}` captures the entire remaining path after a prefix.
- **Pattern precedence**: More specific patterns automatically take priority over less specific ones.

## Real World Context
Before Go 1.22, most teams used gorilla/mux or chi just for method routing and path parameters. Now the standard library covers these use cases, reducing dependencies and improving long-term maintainability.

## Deep Dive

Register routes with method prefixes and path parameters directly on `ServeMux`.

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /users", listUsers)
mux.HandleFunc("POST /users", createUser)
mux.HandleFunc("GET /users/{id}", getUser)
mux.HandleFunc("DELETE /users/{id}", deleteUser)
```

Each route only matches its specified HTTP method. A POST to `/users/{id}` would return 405 Method Not Allowed.

Extract path parameters inside your handler using `r.PathValue`.

```go
func getUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    fmt.Fprintf(w, "User ID: %s", id)
}
```

The value is always a string — parse it to an integer if your ID is numeric.

Wildcard segments capture everything after the prefix, useful for file serving.

```go
mux.HandleFunc("GET /files/{path...}", serveFile)

func serveFile(w http.ResponseWriter, r *http.Request) {
    path := r.PathValue("path")
    // path contains everything after /files/
}
```

This matches `/files/a`, `/files/a/b/c`, and so on.

### Precedence Rules

More specific patterns take precedence:
*   `GET /users/admin` beats `GET /users/{id}` (literal segment beats wildcard)
*   `GET /users/{id}` beats `GET /users/{path...}` (single wildcard beats multi-segment wildcard)
*   Longer patterns beat shorter ones

## Common Pitfalls
1. **Forgetting the method prefix** — `mux.HandleFunc("/users", handler)` matches ALL methods. Add `GET ` prefix to restrict.
2. **Expecting typed path values** — `r.PathValue()` always returns a string. Forgetting to parse it to an int causes subtle bugs.

## Best Practices
1. **Always specify the HTTP method** — Explicit method routing prevents accidental handling of unintended request types.
2. **Use `{path...}` sparingly** — Multi-segment wildcards are greedy and can shadow more specific routes if placed carelessly.

## Summary
- Go 1.22+ ServeMux supports `METHOD /path` routing and `{param}` path parameters natively.
- Use `r.PathValue("name")` to extract dynamic segments from the URL.
- More specific patterns automatically win over less specific ones.

## Code Examples

**A route with a path parameter using Go 1.22+ enhanced ServeMux — r.PathValue extracts the dynamic {id} segment**

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /items/{id}", func(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    fmt.Fprintf(w, "Item ID: %s", id)
})

http.ListenAndServe(":8080", mux)
```


## Resources

- [ServeMux Documentation](https://pkg.go.dev/net/http#ServeMux) — Official Go documentation for the enhanced ServeMux router
- [Routing Enhancements in Go 1.22](https://go.dev/blog/routing-enhancements) — Official blog post explaining the new routing features in Go 1.22

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*