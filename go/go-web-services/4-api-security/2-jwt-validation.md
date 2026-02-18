---
source_course: "go-web-services"
source_lesson: "go-web-services-jwt-validation"
---

# Validating Tokens

```go
func validateToken(tokenString string, secret []byte) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(t *jwt.Token) (any, error) {
        // Validate signing method
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

## Auth Middleware

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

## Code Examples

**Using Claims in Handler**

```go
// Extract user from context
func GetUser(ctx context.Context) *Claims {
    claims, _ := ctx.Value(userClaimsKey).(*Claims)
    return claims
}

// In handler
func handler(w http.ResponseWriter, r *http.Request) {
    user := GetUser(r.Context())
    if user == nil {
        http.Error(w, "Unauthorized", 401)
        return
    }
    fmt.Fprintf(w, "Hello, user %s", user.UserID)
}
```


---

> 📘 *This lesson is part of the [Go for Web & Microservices](https://stanza.dev/courses/go-web-services) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*