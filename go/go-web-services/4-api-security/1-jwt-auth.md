---
source_course: "go-web-services"
source_lesson: "go-web-services-jwt-auth"
---

# JWT Authentication

## Introduction
JSON Web Tokens (JWT) provide stateless authentication for APIs. The server signs a token containing user claims, and clients send it with each request — no session storage required.

## Key Concepts
- **JWT structure**: Three Base64-encoded parts separated by dots: header (algorithm), payload (claims), and signature.
- **Claims**: Key-value pairs embedded in the token, such as `user_id`, `exp` (expiration), and `iat` (issued at).
- **Signing method**: The algorithm used to create the signature (e.g., HS256 for HMAC, RS256 for RSA).
- **Stateless auth**: The server verifies the token signature without a database lookup, making authentication fast and scalable.

## Real World Context
JWTs are the industry standard for API authentication. They eliminate the need for server-side session storage, making horizontal scaling straightforward. Every microservice can independently verify tokens using a shared secret or public key.

## Deep Dive

The JWT authentication flow works as follows:
1.  User logs in with credentials.
2.  Server validates credentials and creates a signed JWT.
3.  Client sends JWT in `Authorization: Bearer <token>` header.
4.  Middleware verifies the signature and extracts claims.

Create a token using the `golang-jwt/jwt` library.

```go
import "github.com/golang-jwt/jwt/v5"

type Claims struct {
    UserID string `json:"user_id"`
    jwt.RegisteredClaims
}

func createToken(userID string, secret []byte) (string, error) {
    claims := Claims{
        UserID: userID,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
        },
    }
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secret)
}
```

The `RegisteredClaims` struct provides standard fields like expiration and issued-at timestamps.

**Security Tip:** Always use HTTPS. Never store sensitive data in the JWT payload — it is Base64 encoded, not encrypted. Anyone can decode and read the claims.

## Common Pitfalls
1. **Storing secrets in the JWT payload** — The payload is only Base64 encoded, not encrypted. Anyone with the token can read it.
2. **Not setting an expiration** — Tokens without `exp` live forever. If compromised, they grant permanent access.

## Best Practices
1. **Keep token lifetimes short** — Use 15-minute access tokens with longer-lived refresh tokens to limit the damage of a stolen token.
2. **Always validate the signing algorithm** — Check that the token's `alg` header matches what you expect to prevent algorithm confusion attacks.

## Summary
- JWTs provide stateless authentication with signed claims — no server-side session storage needed.
- Always set an expiration (`exp`) and keep token lifetimes short.
- Never put sensitive data in the payload — it is readable by anyone with the token.

## Code Examples

**Creating and signing a JWT with HS256 — the signing key must be a byte slice and should be kept secret**

```go
token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
ss, err := token.SignedString(mySigningKey)
```


## Resources

- [golang-jwt/jwt Package](https://pkg.go.dev/github.com/golang-jwt/jwt/v5) — Official documentation for the Go JWT library

---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*