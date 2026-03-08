---
source_course: "rails-api-development"
source_lesson: "rails-api-development-jwt-authentication"
---

# JWT Authentication

## Introduction
JSON Web Tokens (JWT) are a stateless authentication mechanism where the server encodes user data into a signed token. The client stores this token and sends it with each request. Unlike database-backed tokens, JWTs don't require a database lookup on every request.

## Key Concepts
- **JWT (JSON Web Token)**: A self-contained token with three parts: header, payload, and signature.
- **Stateless Authentication**: The server doesn't store session data. All information is encoded in the token.
- **Token Expiration**: JWTs include an `exp` claim that sets when the token becomes invalid.
- **Refresh Tokens**: Long-lived tokens used to obtain new access tokens without re-authenticating.

## Real World Context
JWTs are popular for APIs serving mobile apps and SPAs where session cookies aren't available. They scale well because any server can verify the token without hitting a database. However, they can't be revoked without additional infrastructure.

## Deep Dive
### Setting Up JWT

Add the jwt gem:

```ruby
# Gemfile
gem "jwt"
```

Create a JWT service:

```ruby
# app/services/jwt_service.rb
class JwtService
  SECRET = Rails.application.credentials.secret_key_base
  ALGORITHM = "HS256"

  def self.encode(payload, exp: 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET, ALGORITHM)
  end

  def self.decode(token)
    decoded = JWT.decode(token, SECRET, true, algorithm: ALGORITHM)
    decoded.first.with_indifferent_access
  rescue JWT::DecodeError, JWT::ExpiredSignature
    nil
  end
end
```

The service encodes user data with an expiration and decodes/verifies tokens. It uses the app's secret key base for signing.

### Login Endpoint

```ruby
def create
  user = User.authenticate_by(
    email: params[:email],
    password: params[:password]
  )

  if user
    token = JwtService.encode(user_id: user.id)
    render json: { token: token, expires_in: 24.hours.to_i }
  else
    render json: { error: "Invalid credentials" }, status: :unauthorized
  end
end
```

The login endpoint returns a signed JWT containing the user ID.

### Verifying Tokens

```ruby
class ApplicationController < ActionController::API
  private

  def authenticate_user!
    token = request.headers["Authorization"]&.remove("Bearer ")
    payload = JwtService.decode(token)
    @current_user = User.find_by(id: payload&.dig(:user_id))

    render json: { error: "Unauthorized" }, status: :unauthorized unless @current_user
  end
end
```

Each request extracts the token, decodes it, and finds the user. If the token is expired or invalid, `decode` returns nil.

## Common Pitfalls
1. **No revocation strategy** — JWTs are valid until they expire. If a token is compromised, you can't invalidate it without maintaining a blocklist (which defeats the stateless benefit).
2. **Storing JWTs in localStorage** — Vulnerable to XSS attacks. Use httpOnly cookies for browser clients.

## Best Practices
1. **Set short expiration times** — 15-60 minutes for access tokens, use refresh tokens for longer sessions.
2. **Include only necessary claims** — Don't put sensitive data in the JWT payload. It's encoded, not encrypted.

## Summary
- JWTs are stateless tokens encoding user data with a signature.
- The `jwt` gem handles encoding and decoding with HMAC signing.
- Tokens include expiration claims for automatic invalidation.
- JWTs scale well but can't be individually revoked without a blocklist.
- Use short-lived access tokens with refresh tokens for security.

## Code Examples

**A JWT service class for encoding user data into signed tokens and decoding them with expiration handling**

```ruby
class JwtService
  SECRET = Rails.application.credentials.secret_key_base

  def self.encode(payload, exp: 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET, "HS256")
  end

  def self.decode(token)
    JWT.decode(token, SECRET, true, algorithm: "HS256")
       .first.with_indifferent_access
  rescue JWT::DecodeError, JWT::ExpiredSignature
    nil
  end
end
```


## Resources

- [JWT Ruby Gem](https://github.com/jwt/ruby-jwt) — The ruby-jwt gem documentation for encoding and decoding JSON Web Tokens

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*