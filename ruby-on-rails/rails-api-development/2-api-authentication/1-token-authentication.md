---
source_course: "rails-api-development"
source_lesson: "rails-api-development-token-authentication"
---

# Token-Based Authentication

APIs use tokens instead of sessions since they're stateless. Let's implement a simple token-based auth system.

## How Token Auth Works

1. Client sends credentials (email/password)
2. Server validates and returns a token
3. Client sends token with each request
4. Server validates token and identifies user

## Setting Up the User Model

```ruby
# Migration
class AddAuthTokenToUsers < ActiveRecord::Migration[8.0]
  def change
    add_column :users, :auth_token, :string
    add_index :users, :auth_token, unique: true
  end
end

# app/models/user.rb
class User < ApplicationRecord
  has_secure_password

  before_create :generate_auth_token

  validates :email, presence: true, uniqueness: true

  def regenerate_auth_token
    update(auth_token: generate_token)
  end

  private

  def generate_auth_token
    self.auth_token = generate_token
  end

  def generate_token
    loop do
      token = SecureRandom.hex(32)
      break token unless User.exists?(auth_token: token)
    end
  end
end
```

## Authentication Controller

```ruby
# app/controllers/api/v1/authentication_controller.rb
module Api
  module V1
    class AuthenticationController < BaseController
      skip_before_action :authenticate_user!, only: [:create]

      # POST /api/v1/auth
      def create
        user = User.find_by(email: params[:email])

        if user&.authenticate(params[:password])
          render json: {
            token: user.auth_token,
            user: {
              id: user.id,
              email: user.email,
              name: user.name
            }
          }
        else
          render json: { error: "Invalid credentials" }, status: :unauthorized
        end
      end

      # DELETE /api/v1/auth
      def destroy
        current_user.regenerate_auth_token
        head :no_content
      end
    end
  end
end
```

## Base Controller with Auth

```ruby
# app/controllers/api/v1/base_controller.rb
module Api
  module V1
    class BaseController < ApplicationController
      before_action :authenticate_user!

      private

      def authenticate_user!
        unless current_user
          render json: { error: "Unauthorized" }, status: :unauthorized
        end
      end

      def current_user
        @current_user ||= authenticate_with_token
      end

      def authenticate_with_token
        # Check Authorization header
        header = request.headers["Authorization"]
        return nil unless header

        # Extract token from "Bearer <token>"
        token = header.split(" ").last
        User.find_by(auth_token: token)
      end
    end
  end
end
```

## Client Usage

```bash
# Login
curl -X POST http://localhost:3000/api/v1/auth \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "secret"}'

# Response
{"token": "abc123...", "user": {"id": 1, "email": "user@example.com"}}

# Authenticated request
curl http://localhost:3000/api/v1/articles \
  -H "Authorization: Bearer abc123..."
```

## Routes

```ruby
namespace :api do
  namespace :v1 do
    resource :auth, only: [:create, :destroy], controller: "authentication"
    # or
    post "auth", to: "authentication#create"
    delete "auth", to: "authentication#destroy"
  end
end
```

## Resources

- [Securing Rails Applications](https://guides.rubyonrails.org/security.html) — Rails security best practices

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*