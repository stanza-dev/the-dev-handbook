---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-authorization"
---

# Rails 8 Authentication Generator

## Introduction
Rails 8 introduced a built-in authentication generator that scaffolds a complete session-based authentication system. While designed for full-stack apps, it provides a solid foundation that can be adapted for API token authentication.

## Key Concepts
- **`bin/rails generate authentication`**: The Rails 8 command that generates User model, Session model, controllers, and views for authentication.
- **Session Model**: A database-backed session that tracks authenticated users.
- **Password Reset Flow**: The generator includes password reset functionality with secure tokens.

## Real World Context
Before Rails 8, authentication required either a gem (Devise, Clearance) or building from scratch. The built-in generator provides a maintained, secure starting point that follows Rails conventions. For APIs, you adapt the generated code to use tokens instead of sessions.

## Deep Dive
### Running the Generator

```bash
bin/rails generate authentication
```

This creates:

```
app/models/user.rb              # User model with has_secure_password
app/models/session.rb           # Session model
app/controllers/sessions_controller.rb
app/controllers/passwords_controller.rb
app/mailers/passwords_mailer.rb
db/migrate/..._create_users.rb
db/migrate/..._create_sessions.rb
```

The generated User model:

```ruby
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy

  normalizes :email_address, with: -> (e) { e.strip.downcase }
end
```

Notice `normalizes :email_address` — a Rails 7.1+ feature that automatically strips and downcases emails before saving.

### Adapting for API Use

The generator creates session-based auth. For APIs, adapt it to return tokens:

```ruby
# app/controllers/api/v1/auth_controller.rb
module Api
  module V1
    class AuthController < ApplicationController
      def login
        user = User.authenticate_by(
          email_address: params[:email_address],
          password: params[:password]
        )

        if user
          session = user.sessions.create!
          render json: {
            token: session.id,
            user: { id: user.id, email: user.email_address }
          }, status: :created
        else
          render json: { error: "Invalid credentials" },
                 status: :unauthorized
        end
      end

      def logout
        Current.session&.destroy
        head :no_content
      end
    end
  end
end
```

The session ID serves as the API token. Each login creates a new session, and logout destroys it. This gives you token revocation for free.

### Current Attributes

The generator sets up `Current` for request-scoped attributes:

```ruby
class Current < ActiveSupport::CurrentAttributes
  attribute :session
  delegate :user, to: :session, allow_nil: true
end
```

This lets you access `Current.user` anywhere in the request cycle.

## Common Pitfalls
1. **Using the generator as-is for APIs** — The generated controllers render HTML. You need to adapt them for JSON responses.
2. **Forgetting to run migrations** — The generator creates migration files but doesn't run them automatically.

## Best Practices
1. **Start with the generator, then customize** — It provides secure defaults. Build on them rather than starting from scratch.
2. **Use session-based tokens for revocation** — Database-backed sessions can be individually revoked, unlike stateless JWTs.

## Summary
- `bin/rails generate authentication` scaffolds a complete auth system in Rails 8.
- It creates User and Session models, controllers, and password reset flow.
- Adapt the generated code for API use by returning session IDs as tokens.
- Current attributes provide request-scoped access to the authenticated user.
- Database-backed sessions give you token revocation for free.

## Code Examples

**The User model from Rails 8's auth generator — includes secure password, sessions, and email normalization**

```ruby
# Generated User model (Rails 8)
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy

  normalizes :email_address,
    with: -> (e) { e.strip.downcase }

  validates :email_address,
    presence: true,
    uniqueness: true,
    format: { with: URI::MailTo::EMAIL_REGEXP }
end
```


## Resources

- [Getting Started with Rails](https://guides.rubyonrails.org/getting_started.html) — Rails guide covering the authentication generator and setup

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*