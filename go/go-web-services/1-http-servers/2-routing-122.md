---
source_course: "go-web-services"
source_lesson: "go-web-services-routing-122"
---

# ServeMux Enhancements

Go 1.22 enhanced `http.ServeMux` to support HTTP methods and path parameters.

## Method Routing

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /users", listUsers)
mux.HandleFunc("POST /users", createUser)
mux.HandleFunc("GET /users/{id}", getUser)
mux.HandleFunc("DELETE /users/{id}", deleteUser)
```

## Path Parameters

```go
func getUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    fmt.Fprintf(w, "User ID: %s", id)
}
```

## Wildcards

```go
// Match any path under /files/
mux.HandleFunc("GET /files/{path...}", serveFile)

func serveFile(w http.ResponseWriter, r *http.Request) {
    path := r.PathValue("path")  // Everything after /files/
}
```

## Precedence

More specific patterns take precedence:
*   `GET /users/{id}` beats `GET /users/{name}`
*   Exact paths beat wildcards

## Code Examples

**Path Parameters**

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /items/{id}", func(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    fmt.Fprintf(w, "Item ID: %s", id)
})

http.ListenAndServe(":8080", mux)
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*