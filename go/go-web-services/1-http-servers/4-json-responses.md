---
source_course: "go-web-services"
source_lesson: "go-web-services-json-responses"
---

# JSON Request & Response Handling

## Introduction
Most Go web services communicate using JSON. Go's `encoding/json` package provides `Encoder` and `Decoder` types that stream JSON directly to and from HTTP bodies, making it efficient and straightforward.

## Key Concepts
- **json.NewEncoder(w).Encode(v)**: Writes a Go value as JSON directly to the `ResponseWriter` stream.
- **json.NewDecoder(r.Body).Decode(&v)**: Reads JSON from the request body and populates a Go struct.
- **Struct tags**: Annotations like `json:"name"` control how struct fields map to JSON keys.
- **http.MaxBytesReader**: Wraps the request body to limit its size, preventing denial-of-service via oversized payloads.

## Real World Context
Every REST API needs to serialize responses and deserialize requests. Using `json.Encoder`/`Decoder` (streaming) instead of `json.Marshal`/`Unmarshal` (buffered) avoids unnecessary memory allocations, especially important under high traffic.

## Deep Dive

Sending a JSON response requires setting the Content-Type header and encoding the value.

```go
func getUser(w http.ResponseWriter, r *http.Request) {
    user := User{ID: 1, Name: "Alice"}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```

The encoder writes directly to the `ResponseWriter`, avoiding an intermediate byte buffer.

Parsing a JSON request body follows a similar pattern with the decoder.

```go
func createUser(w http.ResponseWriter, r *http.Request) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    // Validate and process user...
}
```

Always check the decode error — malformed JSON returns a descriptive error message.

For consistent error responses, define an error struct and a helper function.

```go
type ErrorResponse struct {
    Error   string `json:"error"`
    Code    string `json:"code,omitempty"`
    Details any    `json:"details,omitempty"`
}

func writeError(w http.ResponseWriter, status int, msg string) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(ErrorResponse{Error: msg})
}
```

This ensures every error response has the same shape, making client-side error handling predictable.

Limit request body size to prevent abuse.

```go
r.Body = http.MaxBytesReader(w, r.Body, 1<<20)  // 1MB limit
```

If the body exceeds the limit, subsequent reads return an error.

## Common Pitfalls
1. **Forgetting `Content-Type: application/json`** — Without this header, clients may not parse the response as JSON, leading to confusing errors.
2. **Using `json.Unmarshal` instead of `json.NewDecoder`** — `Unmarshal` requires reading the entire body into memory first with `io.ReadAll`, which is wasteful for large payloads.

## Best Practices
1. **Always limit request body size** — Use `http.MaxBytesReader` to prevent memory exhaustion from oversized requests.
2. **Define a consistent error response struct** — A uniform error shape across all endpoints simplifies client error handling.

## Summary
- Use `json.NewEncoder(w).Encode()` for responses and `json.NewDecoder(r.Body).Decode()` for requests.
- Always set `Content-Type: application/json` before writing.
- Limit body size with `http.MaxBytesReader` to prevent abuse.

## Code Examples

**A JSON REST API using Go 1.22+ method routing — separate handlers for GET and POST without switch statements**

```go
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("GET /users", func(w http.ResponseWriter, r *http.Request) {
        users := []User{{ID: 1, Name: "Alice", Email: "alice@example.com"}}
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(users)
    })
    mux.HandleFunc("POST /users", func(w http.ResponseWriter, r *http.Request) {
        var user User
        if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusCreated)
        json.NewEncoder(w).Encode(user)
    })
    http.ListenAndServe(":8080", mux)
}
```


## Resources

- [encoding/json Package](https://pkg.go.dev/encoding/json) — Official Go documentation for JSON encoding and decoding
- [JSON and Go](https://go.dev/blog/json) — Official blog post explaining JSON handling patterns in Go

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*