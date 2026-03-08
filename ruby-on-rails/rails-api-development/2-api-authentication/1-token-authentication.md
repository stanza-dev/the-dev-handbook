---
source_course: "rails-api-development"
source_lesson: "rails-api-development-token-authentication"
---

# API Authentication Fundamentals

## Introduction
APIs cannot use browser cookies for authentication. Instead, they rely on tokens — unique strings that clients include in request headers to prove their identity. Rails 8 provides built-in tools for implementing token authentication.

## Key Concepts
- **Token Authentication**: Clients send a token in the `Authorization` header with each request.
- **`has_secure_password`**: A Rails method that adds bcrypt password hashing to a model.
- **`has_secure_token`**: Generates a unique token for a model attribute.
- **`authenticate_by`**: A Rails method for secure password checking that prevents timing attacks.

## Real World Context
Every API with user accounts needs authentication. Mobile apps, SPAs, and third-party integrations all send tokens with requests. The token proves the client is who they claim to be without sending credentials on every request.

## Deep Dive
### Setting Up User Model

```ruby
# Generate user model with password digest
rails generate model User email:string:uniq password_digest:string api_token:string:uniq
```

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password
  has_secure_token :api_token

  validates :email, presence: true, uniqueness: true
  validates :email, format: { with: URI::MailTo::EMAIL_REGEXP }
end
```

`has_secure_password` adds `password` and `password_confirmation` virtual attributes and stores a bcrypt hash in `password_digest`. `has_secure_token :api_token` generates a unique 24-character token on creation.

### Authentication Controller

```ruby
class Api::V1::SessionsController < ApplicationController
  def create
    user = User.authenticate_by(
      email: params[:email],
      password: params[:password]
    )

    if user
      render json: { token: user.api_token, user: user.as_json(only: [:id, :email]) }
    else
      render json: { error: "Invalid email or password" }, status: :unauthorized
    end
  end
end
```

`authenticate_by` is a Rails method that safely looks up the user and verifies the password in constant time, preventing timing attacks that could reveal whether an email exists.

### Token Verification

```ruby
class ApplicationController < ActionController::API
  private

  def authenticate_user!
    token = request.headers["Authorization"]&.remove("Bearer ")
    @current_user = User.find_by(api_token: token)

    render json: { error: "Unauthorized" }, status: :unauthorized unless @current_user
  end

  def current_user
    @current_user
  end
end
```

The `authenticate_user!` method extracts the Bearer token from the Authorization header and finds the matching user. Controllers that need authentication call this as a `before_action`.

### Protecting Endpoints

```ruby
class Api::V1::ProductsController < ApplicationController
  before_action :authenticate_user!
  before_action :set_product, only: [:show, :update, :destroy]

  def index
    render json: current_user.products
  end
end
```

The `before_action` ensures every request to this controller is authenticated.

## Common Pitfalls
1. **Storing tokens in plain text** — While `has_secure_token` generates cryptographically secure tokens, consider hashing stored tokens for defense in depth.
2. **Not using `authenticate_by`** — Manual email lookup + `authenticate` is vulnerable to timing attacks that reveal which emails exist in your system.

## Best Practices
1. **Use `authenticate_by`** — It's the secure, Rails-recommended way to verify credentials.
2. **Always use Bearer token format** — `Authorization: Bearer <token>` is the standard format.

## Summary
- APIs use token-based authentication via the `Authorization: Bearer <token>` header.
- `has_secure_password` handles password hashing with bcrypt.
- `has_secure_token` generates unique API tokens.
- `authenticate_by` prevents timing attacks during credential verification.
- Protect endpoints with `before_action :authenticate_user!`.

## Code Examples

**The authenticate_by method — Rails' secure credential verification that prevents timing attacks**

```ruby
# Secure login with authenticate_by (Rails 7.1+)
user = User.authenticate_by(
  email: params[:email],
  password: params[:password]
)
# Returns the user if credentials match, nil otherwise.
# Uses constant-time comparison to prevent timing attacks.
# Safer than: User.find_by(email: email)&.authenticate(password)
```


## Resources

- [Active Model SecurePassword](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) — API docs for has_secure_password and authenticate_by

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*