---
source_course: "rails-foundations"
source_lesson: "rails-foundations-rails-8-authentication"
---

# Rails Authentication Generator

## Introduction

Rails provides a built-in authentication generator that creates a complete, production-ready authentication system in seconds. Instead of installing third-party gems, you get a clean, understandable codebase you fully own.

## Key Concepts

- **Authentication generator**: `bin/rails generate authentication` scaffolds a complete auth system with models, controllers, views, and mailers.
- **Session model**: A database-backed session that tracks user sessions across devices.
- **`Current` attributes**: A thread-safe way to access the current user and session throughout the request lifecycle.
- **Password reset flow**: A secure token-based flow for resetting forgotten passwords, generated automatically.

## Real World Context

The authentication generator gives you exactly what most applications need: user registration, login, logout, and password reset. It is designed to be simple, secure, and customizable. Unlike third-party gems like Devise, you own all the generated code and can modify it freely.

## Deep Dive

### Generating Authentication

```bash
bin/rails generate authentication
```

This creates:
- User model with email and password
- Session model for managing sessions
- Controllers for registration, login, password reset
- Views for all authentication forms
- Mailer for password reset emails

### Key Components

```ruby
# User model
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy
  validates :email, presence: true, uniqueness: true
  normalizes :email, with: -> e { e.strip.downcase }
end

# Session model
class Session < ApplicationRecord
  belongs_to :user
end

# Current attributes
class Current < ActiveSupport::CurrentAttributes
  attribute :session, :user_agent, :ip_address
  delegate :user, to: :session, allow_nil: true
end
```

### Using Authentication

```ruby
class ArticlesController < ApplicationController
  before_action :require_authentication, except: [:index, :show]

  def create
    @article = Current.user.articles.build(article_params)
  end
end
```

### In Views

```erb
<% if Current.user %>
  <p>Hello, <%= Current.user.email %></p>
  <%= button_to "Log out", session_path, method: :delete %>
<% else %>
  <%= link_to "Log in", new_session_path %>
<% end %>
```

### Session Management

```ruby
user.sessions               # All active sessions
session.destroy              # Log out that device
user.sessions.destroy_all   # Log out everywhere
```

## Common Pitfalls

- **Not running the migration after generating**: The generator creates migrations that must be run with `bin/rails db:migrate`.
- **Confusing `Current.user` with `current_user`**: The generated code uses `Current.user` via CurrentAttributes, not a `current_user` helper method.
- **Overcomplicating the generated code**: The generated authentication is intentionally simple. Resist the urge to add complexity before you need it.

## Best Practices

- Use the built-in generator before reaching for third-party authentication gems.
- Review all generated code to understand how authentication works.
- Customize the generated views to match your application's design.

## Summary

- `bin/rails generate authentication` creates a complete auth system.
- It generates User and Session models, controllers, views, and a password reset mailer.
- `Current.user` provides thread-safe access to the logged-in user anywhere in the request.
- Session tracking allows users to manage and revoke sessions across devices.
- The generated code is fully owned by your application and freely customizable.

## Code Examples

**The Rails authentication generator creates a complete system. Current.user provides thread-safe access to the logged-in user via CurrentAttributes.**

```ruby
# Generate the authentication system
# bin/rails generate authentication

# Access the current user anywhere
Current.user         # The logged-in user
Current.session      # The current session

# Protect controllers
class ArticlesController < ApplicationController
  before_action :require_authentication, except: [:index, :show]
end
```


## Resources

- [Rails Release Notes - Authentication](https://guides.rubyonrails.org/8_0_release_notes.html) — Rails authentication generator documentation

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*