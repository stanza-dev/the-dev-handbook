---
source_course: "go-web-services"
source_lesson: "go-web-services-net-http-basics"
---

# HTTP Servers

Go's standard library is production-ready. You often don't need a framework like Gin or Echo.

## Basic Server

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Handlers
A handler is any type implementing `ServeHTTP(w, r)`. Functions can be converted using `http.HandlerFunc`.

```go
type myHandler struct{}

func (h *myHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello from handler"))
}

http.Handle("/", &myHandler{})
```

## Code Examples

**Simple Server**

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*