---
source_course: "go-web-services"
source_lesson: "go-web-services-jwt-validation"
---

# JWT Validation Middleware

## Introduction
Validating JWTs on every request is the core of stateless API authentication. A middleware intercepts requests, verifies the token signature and expiration, and injects the user claims into the request context.

## Key Concepts
- **jwt.ParseWithClaims**: Parses a token string, validates it against the signing method, and populates the claims struct.
- **Key function**: A callback that receives the token and returns the signing key — this is where you validate the algorithm.
- **Context injection**: Storing verified claims in `context.WithValue` so downstream handlers can access user information.
- **Bearer scheme**: The standard format for sending JWTs: `Authorization: Bearer <token>`.

## Real World Context
Every protected API endpoint needs JWT validation. Implementing it as middleware means you write the logic once and apply it to any route group. This is the standard pattern used in production Go services.

## Deep Dive

The validation function parses the token, checks the signing method, and returns the claims.

```go
func validateToken(tokenString string, secret []byte) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(t *jwt.Token) (any, error) {
        if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", t.Header["alg"])
        }
        return secret, nil
    })

    if err != nil {
        return nil, err
    }

    claims, ok := token.Claims.(*Claims)
    if !ok || !token.Valid {
        return nil, errors.New("invalid token")
    }

    return claims, nil
}
```

Validating the signing method in the key function prevents algorithm confusion attacks.

The middleware extracts the token from the Authorization header and injects claims into the context.

```go
func AuthMiddleware(secret []byte) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            header := r.Header.Get("Authorization")
            if !strings.HasPrefix(header, "Bearer ") {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }

            tokenString := strings.TrimPrefix(header, "Bearer ")
            claims, err := validateToken(tokenString, secret)
            if err != nil {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }

            ctx := context.WithValue(r.Context(), userClaimsKey, claims)
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}
```

Downstream handlers retrieve claims from the context without re-validating the token.

## Common Pitfalls
1. **Not validating the signing algorithm** — An attacker could craft a token with `alg: none` and bypass signature verification entirely.
2. **Returning detailed error messages** — Telling the client why validation failed (expired, invalid signature) leaks information. Always return a generic 401.

## Best Practices
1. **Use a typed context key** — Avoid string keys for context values. Use an unexported struct type to prevent collisions.
2. **Return early on auth failure** — Never call `next.ServeHTTP` after detecting an invalid token.

## Summary
- Parse and validate JWTs with `jwt.ParseWithClaims`, always checking the signing algorithm.
- Inject verified claims into the request context so handlers can access user identity.
- Return generic 401 errors — never expose why token validation failed.

## Code Examples

**Extracting verified user claims from the request context — the type assertion is safe because the middleware already validated the token**

```go
func GetUser(ctx context.Context) *Claims {
    claims, _ := ctx.Value(userClaimsKey).(*Claims)
    return claims
}

func handler(w http.ResponseWriter, r *http.Request) {
    user := GetUser(r.Context())
    if user == nil {
        http.Error(w, "Unauthorized", 401)
        return
    }
    fmt.Fprintf(w, "Hello, user %s", user.UserID)
}
```


## Resources

- [golang-jwt/jwt Package](https://pkg.go.dev/github.com/golang-jwt/jwt/v5) — Official documentation covering ParseWithClaims and token validation

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*