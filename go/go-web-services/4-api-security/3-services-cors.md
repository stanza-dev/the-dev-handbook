---
source_course: "go-web-services"
source_lesson: "go-web-services-cors"
---

# CORS & Security Headers

## Introduction
Browsers enforce the Same-Origin Policy, blocking requests from one domain to another by default. CORS (Cross-Origin Resource Sharing) is the mechanism that allows your API to accept requests from authorized frontend origins.

## Key Concepts
- **Same-Origin Policy**: A browser security feature that prevents JavaScript on one origin from making requests to a different origin.
- **Preflight request**: An OPTIONS request the browser sends before the actual request to check if the server allows the cross-origin call.
- **Access-Control-Allow-Origin**: The response header that specifies which origins are permitted to access the resource.
- **Security headers**: Additional headers (X-Content-Type-Options, HSTS, X-Frame-Options) that harden the response against common web attacks.

## Real World Context
If your React frontend at `app.example.com` calls your Go API at `api.example.com`, the browser blocks it unless your API returns proper CORS headers. This is one of the first issues developers hit when connecting a frontend to a separate backend.

## Deep Dive

A CORS middleware sets the required headers and handles preflight OPTIONS requests.

```go
func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "https://example.com")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")

        if r.Method == http.MethodOptions {
            w.WriteHeader(http.StatusOK)
            return
        }

        next.ServeHTTP(w, r)
    })
}
```

The preflight response tells the browser which methods and headers are allowed. The browser then sends the actual request.

Security headers protect against common web attacks.

```go
func securityHeaders(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")

        next.ServeHTTP(w, r)
    })
}
```

These headers prevent MIME sniffing, clickjacking, XSS, and enforce HTTPS.

## Common Pitfalls
1. **Using `*` for Allow-Origin in production** — A wildcard allows any website to call your API, which is a security risk for authenticated endpoints.
2. **Forgetting to handle OPTIONS preflight** — Without handling the preflight request, the browser never sends the actual request.

## Best Practices
1. **Whitelist specific origins** — Only allow known frontend domains, not `*`.
2. **Use a CORS library for complex configurations** — Libraries like `github.com/rs/cors` handle edge cases around credentials, caching, and multiple origins.

## Summary
- CORS headers tell browsers which origins, methods, and headers are allowed for cross-origin requests.
- Always handle OPTIONS preflight requests explicitly.
- Add security headers (HSTS, X-Frame-Options, nosniff) to every response.

## Code Examples

**Using the rs/cors library for production CORS configuration — supports credentials, multiple origins, and automatic preflight handling**

```go
import "github.com/rs/cors"

c := cors.New(cors.Options{
    AllowedOrigins:   []string{"https://example.com"},
    AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE"},
    AllowedHeaders:   []string{"Authorization", "Content-Type"},
    AllowCredentials: true,
})

handler := c.Handler(mux)
```


## Resources

- [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) — MDN reference explaining CORS mechanics and preflight requests

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*