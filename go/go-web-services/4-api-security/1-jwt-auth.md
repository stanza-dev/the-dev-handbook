---
source_course: "go-web-services"
source_lesson: "go-web-services-jwt-auth"
---

# Stateless Auth

JSON Web Tokens (JWT) allow stateless authentication. The server verifies the token signature without checking the DB.

## The Flow
1.  User logs in.
2.  Server creates a signed JWT.
3.  Client sends JWT in `Authorization: Bearer <token>` header.
4.  Middleware verifies the signature.

## Creating a Token

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

**Security Tip:** Always use HTTPS. Never store sensitive data in the JWT payload (it is base64 encoded, not encrypted).

## Code Examples

**Signing a Token**

```go
token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
ss, err := token.SignedString(mySigningKey)
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*