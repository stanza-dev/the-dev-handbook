---
source_course: "rails-foundations"
source_lesson: "rails-foundations-rails-8-authentication"
---

# Rails 8 Authentication Generator

Rails 8 introduces a built-in authentication generator that creates a complete, production-ready authentication system.

## Generating Authentication

```bash
bin/rails generate authentication
```

This creates:
- User model with email and password
- Session model for managing sessions
- Controllers for registration, login, password reset
- Views for all authentication forms
- Mailer for password reset emails

## Generated Files

```
app/
├── controllers/
│   ├── sessions_controller.rb
│   ├── registrations_controller.rb
│   └── passwords_controller.rb
├── models/
│   ├── user.rb
│   ├── session.rb
│   └── current.rb
├── views/
│   ├── sessions/
│   ├── registrations/
│   └── passwords/
└── mailers/
    └── passwords_mailer.rb
```

## Key Components

### The User Model

```ruby
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy

  validates :email, presence: true, uniqueness: true
  normalizes :email, with: -> e { e.strip.downcase }
end
```

### The Session Model

```ruby
class Session < ApplicationRecord
  belongs_to :user

  before_create do
    self.user_agent = Current.user_agent
    self.ip_address = Current.ip_address
  end
end
```

### The Current Model

Rails 8 uses `Current` attributes for request-local storage:

```ruby
class Current < ActiveSupport::CurrentAttributes
  attribute :session, :user_agent, :ip_address

  delegate :user, to: :session, allow_nil: true
end
```

## Using Authentication

### Protect Controllers

```ruby
class ArticlesController < ApplicationController
  before_action :require_authentication, except: [:index, :show]

  def create
    @article = Current.user.articles.build(article_params)
    # ...
  end
end
```

### Access Current User

```ruby
# In controllers and views
Current.user        # The logged-in user
Current.session     # The current session

# Check if authenticated
if Current.user
  # User is logged in
end
```

### In Views

```erb
<% if Current.user %>
  <p>Hello, <%= Current.user.email %></p>
  <%= button_to "Log out", session_path, method: :delete %>
<% else %>
  <%= link_to "Log in", new_session_path %>
  <%= link_to "Sign up", new_registration_path %>
<% end %>
```

## Session Management

The generator creates session tracking:

```ruby
# View all user sessions
user.sessions  # All active sessions

# Revoke a session
session.destroy  # Log out that device

# Revoke all sessions (log out everywhere)
user.sessions.destroy_all
```

## Password Reset Flow

The generator includes password reset:

1. User requests reset at `/passwords/new`
2. Email sent with secure token
3. User clicks link to `/passwords/:token/edit`
4. User sets new password

```ruby
# Manually send password reset
user = User.find_by(email: params[:email])
PasswordsMailer.reset(user).deliver_later
```

This authentication system is simple, secure, and ready for production use.

## Resources

- [Rails 8 Release Notes - Authentication](https://guides.rubyonrails.org/8_0_release_notes.html) — Rails 8 authentication generator documentation

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*