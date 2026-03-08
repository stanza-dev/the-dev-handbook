---
source_course: "rails-security-hardening"
source_lesson: "rails-security-session-security"
---

# Session Security

## Introduction
Sessions maintain user state across HTTP requests. A compromised session gives an attacker full access to a user's account. Proper session configuration is essential for authentication security.

## Key Concepts
- **Session Fixation**: An attack where an attacker sets a known session ID before the victim logs in, then uses that ID to access the victim's account.
- **Session Hijacking**: Stealing an active session ID through XSS, network sniffing, or other means.
- **HttpOnly Flag**: A cookie attribute that prevents JavaScript from reading the cookie, mitigating XSS-based session theft.
- **Current Model**: A Rails 8 pattern using `Current` thread-local attributes to access the authenticated user throughout the request.

## Real World Context
Session security affects every authenticated request in your application. Rails 8's generated authentication uses the `Current` model pattern and an `Authentication` concern that handle session management with built-in best practices.

## Deep Dive

### Session Configuration

Configure session cookies with security flags:

```ruby
# config/initializers/session_store.rb
Rails.application.config.session_store :cookie_store,
  key: '_myapp_session',
  secure: Rails.env.production?,    # HTTPS only in production
  httponly: true,                   # No JavaScript access
  same_site: :lax                   # CSRF protection
```

The `httponly` flag prevents XSS attacks from reading the session cookie. The `secure` flag ensures cookies are only sent over HTTPS. The `same_site: :lax` flag prevents the cookie from being sent in cross-site requests.

### Rails 8 Authentication Concern

Rails 8's generated `Authentication` concern provides session management:

```ruby
# app/controllers/concerns/authentication.rb
module Authentication
  extend ActiveSupport::Concern

  included do
    before_action :require_authentication
    helper_method :authenticated?
  end

  private

  def authenticated?
    Current.session.present?
  end

  def require_authentication
    resume_session || request_authentication
  end

  def resume_session
    Current.session = find_session_by_cookie
  end

  def find_session_by_cookie
    Session.find_by(id: cookies.signed[:session_id]) if cookies.signed[:session_id]
  end
end
```

This concern uses signed cookies and a `Session` model to manage authentication state.

### The Current Model Pattern

Rails 8 uses `Current` to provide request-scoped access to the authenticated user:

```ruby
# app/models/current.rb
class Current < ActiveSupport::CurrentAttributes
  attribute :session
  delegate :user, to: :session, allow_nil: true
end

# Access anywhere in the request:
Current.user  # => the authenticated user or nil
```

The `Current` model uses thread-local storage and is automatically reset between requests.

### Session Fixation Prevention

Always regenerate the session ID after login:

```ruby
class SessionsController < ApplicationController
  def create
    user = User.authenticate_by(
      email_address: params[:email_address],
      password: params[:password]
    )

    if user
      reset_session  # Regenerate session ID
      session = user.sessions.create!
      cookies.signed.permanent[:session_id] = { value: session.id, httponly: true }
      redirect_to root_path
    else
      redirect_to new_session_path, alert: 'Invalid credentials'
    end
  end

  def destroy
    Current.session.destroy
    cookies.delete(:session_id)
    redirect_to root_path, notice: 'Logged out'
  end
end
```

Calling `reset_session` before setting the authenticated session prevents an attacker from using a pre-set session ID.

## Common Pitfalls
1. **Not regenerating session ID on login** — Without `reset_session`, session fixation attacks allow attackers to hijack the authenticated session.
2. **Storing sensitive data in the session** — Session data is stored in cookies by default. Never put passwords, credit card numbers, or other secrets in the session.

## Best Practices
1. **Use Rails 8's Authentication concern** — It follows all best practices by default, including signed cookies and the Current model pattern.
2. **Set all security flags on session cookies** — `httponly`, `secure`, and `same_site` should all be configured.

## Summary
- Configure session cookies with `httponly`, `secure`, and `same_site` flags.
- Rails 8's authentication generator provides session management with best practices.
- The `Current` model gives request-scoped access to the authenticated user.
- Always call `reset_session` before creating an authenticated session.
- Never store sensitive data in session cookies.

## Code Examples

**Rails 8 authentication pattern — Current model for request-scoped user access and session fixation prevention**

```ruby
# Rails 8 Current model pattern
class Current < ActiveSupport::CurrentAttributes
  attribute :session
  delegate :user, to: :session, allow_nil: true
end

# Access the authenticated user anywhere:
# Current.user => the logged-in user or nil

# Session fixation prevention in login:
def create
  if user = User.authenticate_by(email_address: params[:email], password: params[:password])
    reset_session  # Prevent session fixation
    session = user.sessions.create!
    cookies.signed.permanent[:session_id] = { value: session.id, httponly: true }
  end
end
```


## Resources

- [Rails Security Guide — Sessions](https://guides.rubyonrails.org/security.html#sessions) — Official Rails guide on session security best practices

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*