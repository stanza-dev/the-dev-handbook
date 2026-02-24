---
source_course: "go-web-services"
source_lesson: "go-web-services-net-http-basics"
---

# The net/http Package

## Introduction
Go ships with a production-grade HTTP server in its standard library. Unlike most languages, you rarely need a third-party framework to build robust web services in Go.

## Key Concepts
- **http.Handler**: An interface requiring a single method `ServeHTTP(w, r)` — the foundation of all request handling in Go.
- **http.HandleFunc**: A convenience function that registers a plain function as a handler on the default ServeMux.
- **http.ResponseWriter**: The interface used to construct and send an HTTP response back to the client.
- **http.ListenAndServe**: Starts an HTTP server on a given address, blocking until an error occurs.

## Real World Context
Most Go microservices in production use the standard library directly. Understanding `net/http` is essential because even frameworks like Gin and Echo build on top of it. Knowing the primitives lets you debug issues, write middleware, and avoid unnecessary dependencies.

## Deep Dive

The simplest HTTP server in Go uses `http.HandleFunc` to register a handler function and `http.ListenAndServe` to start the server.

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Passing `nil` to `ListenAndServe` uses the default ServeMux. Every incoming request is dispatched to the matching handler.

For more control, implement the `http.Handler` interface directly. Any type with a `ServeHTTP(w, r)` method satisfies the interface.

```go
type myHandler struct{}

func (h *myHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello from handler"))
}

func main() {
    http.Handle("/", &myHandler{})
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

The `http.HandlerFunc` type adapter lets you convert any function with the right signature into an `http.Handler`, bridging the gap between functions and interfaces.

## Common Pitfalls
1. **Forgetting to call `log.Fatal` on `ListenAndServe`** — If the server fails to start (e.g., port in use), the error is silently discarded without `log.Fatal`.
2. **Writing after `WriteHeader`** — Calling `w.Write()` implicitly sends a 200 status. If you call `w.WriteHeader(404)` after `w.Write()`, the status code is ignored.

## Best Practices
1. **Use `http.NewServeMux()` instead of the default mux** — The default mux is a global variable; creating your own avoids routing collisions in tests and large applications.
2. **Always handle the `ListenAndServe` error** — Wrap it in `log.Fatal` or a structured logger to catch bind failures immediately.

## Summary
- Go's `net/http` package provides a complete HTTP server without external dependencies.
- Handlers implement `ServeHTTP(w, r)` or use `http.HandleFunc` for convenience.
- `http.ListenAndServe` starts the server and blocks until failure.

## Code Examples

**A minimal HTTP server using http.HandleFunc — the default ServeMux handles routing when nil is passed to ListenAndServe**

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```


## Resources

- [net/http Package](https://pkg.go.dev/net/http) — Official Go documentation for the net/http package
- [Writing Web Applications](https://go.dev/doc/articles/wiki) — Official Go tutorial on building a web application from scratch

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*