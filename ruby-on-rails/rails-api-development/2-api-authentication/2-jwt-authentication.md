---
source_course: "rails-api-development"
source_lesson: "rails-api-development-jwt-authentication"
---

# JWT Authentication

JSON Web Tokens (JWTs) are a popular, self-contained token format that includes user data and expiration.

## What is JWT?

A JWT has three parts separated by dots:
- **Header**: Algorithm and token type
- **Payload**: Data (claims) like user_id and expiration
- **Signature**: Verifies the token hasn't been tampered with

```
eyJhbGciOiJIUzI1NiJ9.          # Header
eyJ1c2VyX2lkIjoxLCJleHAiOjE2...# Payload
.dBjftJeZ4CVP-mB92K27uhbUJU1p1... # Signature
```

## Setup

Add the `jwt` gem:

```ruby
# Gemfile
gem 'jwt'
```

## JWT Service

```ruby
# app/services/json_web_token.rb
class JsonWebToken
  SECRET_KEY = Rails.application.credentials.secret_key_base

  def self.encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET_KEY)
  end

  def self.decode(token)
    decoded = JWT.decode(token, SECRET_KEY)[0]
    HashWithIndifferentAccess.new(decoded)
  rescue JWT::ExpiredSignature
    raise AuthenticationError, "Token has expired"
  rescue JWT::DecodeError
    raise AuthenticationError, "Invalid token"
  end
end

# app/errors/authentication_error.rb
class AuthenticationError < StandardError; end
```

## Authentication Controller

```ruby
module Api
  module V1
    class AuthenticationController < BaseController
      skip_before_action :authenticate_user!, only: [:create]

      def create
        user = User.find_by(email: params[:email])

        if user&.authenticate(params[:password])
          token = JsonWebToken.encode(user_id: user.id)
          exp_time = 24.hours.from_now

          render json: {
            token: token,
            exp: exp_time.strftime("%Y-%m-%d %H:%M"),
            user: UserSerializer.new(user)
          }
        else
          render json: { error: "Invalid credentials" }, status: :unauthorized
        end
      end

      # Refresh token endpoint
      def refresh
        token = JsonWebToken.encode(user_id: current_user.id)
        render json: { token: token }
      end
    end
  end
end
```

## Base Controller

```ruby
module Api
  module V1
    class BaseController < ApplicationController
      before_action :authenticate_user!

      rescue_from AuthenticationError, with: :handle_auth_error

      private

      def authenticate_user!
        render json: { error: "Unauthorized" }, status: :unauthorized unless current_user
      end

      def current_user
        @current_user ||= find_user_from_token
      end

      def find_user_from_token
        header = request.headers["Authorization"]
        return nil unless header

        token = header.split(" ").last
        decoded = JsonWebToken.decode(token)
        User.find_by(id: decoded[:user_id])
      end

      def handle_auth_error(exception)
        render json: { error: exception.message }, status: :unauthorized
      end
    end
  end
end
```

## Token Refresh Strategy

```ruby
# Include refresh token for longer sessions
def create
  user = User.find_by(email: params[:email])

  if user&.authenticate(params[:password])
    access_token = JsonWebToken.encode(
      { user_id: user.id },
      15.minutes.from_now  # Short-lived
    )

    refresh_token = JsonWebToken.encode(
      { user_id: user.id, type: "refresh" },
      7.days.from_now  # Long-lived
    )

    render json: {
      access_token: access_token,
      refresh_token: refresh_token,
      expires_in: 900  # seconds
    }
  else
    render json: { error: "Invalid credentials" }, status: :unauthorized
  end
end
```

## Resources

- [JWT.io](https://jwt.io/introduction) — Introduction to JSON Web Tokens

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*