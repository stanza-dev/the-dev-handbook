---
source_course: "rails-foundations"
source_lesson: "rails-foundations-sessions-cookies"
---

# Sessions and Cookies

## Introduction

HTTP is stateless — each request is independent with no memory of previous ones. Sessions and cookies solve this by letting you persist data across requests, enabling features like user login and shopping carts.

## Key Concepts

- **Session**: Server-side storage that persists data across requests for a specific user. In Rails, accessed via the `session` hash.
- **Cookie**: A small piece of data stored in the browser and sent with every request. Rails uses an encrypted cookie as the default session store.
- **Flash messages**: Special session data that persists for exactly one request, used for success/error notifications after redirects.
- **CSRF protection**: Rails automatically includes an authenticity token to prevent cross-site request forgery attacks.

## Real World Context

Every logged-in experience on the web relies on sessions. When you log into a website and it remembers you across page loads, that is sessions at work. Understanding the difference between sessions (secure, server-validated) and cookies (client-stored, readable) is essential for building secure applications.

## Deep Dive

### Using Sessions

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])
    if user&.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to root_path, notice: "Logged in!"
    else
      flash.now[:alert] = "Invalid email or password"
      render :new, status: :unprocessable_entity
    end
  end

  def destroy
    session[:user_id] = nil
    redirect_to root_path, notice: "Logged out!"
  end
end
```

### Accessing Current User

```ruby
class ApplicationController < ActionController::Base
  helper_method :current_user, :logged_in?

  private
  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end

  def logged_in?
    current_user.present?
  end

  def require_login
    unless logged_in?
      redirect_to login_path, alert: "Please log in first"
    end
  end
end
```

### Flash Messages

```ruby
flash[:notice] = "Article saved!"   # For next request
flash.now[:alert] = "Invalid data"  # For current request (when rendering)
```

```erb
<% flash.each do |type, message| %>
  <div class="alert alert-<%= type %>"><%= message %></div>
<% end %>
```

### Sessions vs Cookies

| Sessions | Cookies |
|----------|----------|
| Stored server-side (encrypted cookie) | Stored in browser |
| More secure | Less secure |
| Limited to request lifecycle | Can persist for months |
| Use for: user ID, auth data | Use for: preferences, theme |

### Using Cookies Directly

```ruby
cookies[:theme] = "dark"
cookies.signed[:user_id] = user.id       # Tamper-proof
cookies.encrypted[:secret] = "sensitive"  # Unreadable
cookies.delete(:theme)
```

## Common Pitfalls

- **Storing sensitive data in plain cookies**: Use `cookies.signed` or `cookies.encrypted` for anything sensitive.
- **Using `flash` instead of `flash.now` when rendering**: `flash` persists to the next request, so using it with `render` shows the message twice.
- **Not using `reset_session` after login**: To prevent session fixation attacks, reset the session after successful authentication.

## Best Practices

- Store only the user ID in the session, not the entire user object.
- Use `helper_method` to make `current_user` available in views.
- Always use `flash.now` when rendering (not redirecting).

## Summary

- Sessions persist data across HTTP requests using encrypted cookies by default.
- Store `user_id` in the session for authentication; look up the user on each request.
- Flash messages display notifications after redirects; use `flash.now` when rendering.
- Use `cookies.signed` or `cookies.encrypted` for sensitive cookie data.
- The `current_user` pattern with memoization is the standard way to access the logged-in user.

## Code Examples

**The current_user pattern uses the session to store a user ID and memoizes the database lookup, making the logged-in user available across controllers and views.**

```ruby
class ApplicationController < ActionController::Base
  helper_method :current_user, :logged_in?

  private
  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end

  def logged_in?
    current_user.present?
  end
end
```


## Resources

- [Action Controller Overview - Session](https://guides.rubyonrails.org/action_controller_overview.html#session) — Official guide to sessions in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*