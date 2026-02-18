---
source_course: "rails-foundations"
source_lesson: "rails-foundations-sessions-cookies"
---

# Sessions and Cookies

HTTP is stateless - each request is independent. Sessions let you persist data across requests, enabling features like user login.

## How Sessions Work

1. User logs in with credentials
2. Server creates a session and stores user ID
3. Server sends a session cookie to browser
4. Browser sends cookie with every request
5. Server reads cookie to identify user

## Using Sessions in Rails

Rails provides a `session` hash that works like a Hash:

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])

    if user&.authenticate(params[:password])
      # Store user ID in session
      session[:user_id] = user.id
      redirect_to root_path, notice: "Logged in!"
    else
      flash.now[:alert] = "Invalid email or password"
      render :new, status: :unprocessable_entity
    end
  end

  def destroy
    # Clear the session
    session[:user_id] = nil
    # Or clear everything: reset_session
    redirect_to root_path, notice: "Logged out!"
  end
end
```

## Accessing Current User

Create a helper method in ApplicationController:

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

Use in views:

```erb
<% if logged_in? %>
  <p>Welcome, <%= current_user.name %>!</p>
  <%= link_to "Log out", logout_path, data: { turbo_method: :delete } %>
<% else %>
  <%= link_to "Log in", login_path %>
<% end %>
```

## Flash Messages

Flash stores messages for the next request:

```ruby
# Set flash for next request
flash[:notice] = "Article saved!"
flash[:alert] = "Something went wrong"
redirect_to articles_path

# Set flash for current request (when rendering)
flash.now[:alert] = "Invalid data"
render :new
```

Display in layout:

```erb
<% flash.each do |type, message| %>
  <div class="alert alert-<%= type %>">
    <%= message %>
  </div>
<% end %>
```

## Cookies vs Sessions

| Sessions | Cookies |
|----------|----------|
| Stored server-side | Stored in browser |
| More secure | Less secure |
| Limited to request | Can persist for months |
| Use for: user ID | Use for: preferences |

### Using Cookies Directly

```ruby
# Set a cookie
cookies[:theme] = "dark"

# Set with options
cookies[:remember_token] = {
  value: user.remember_token,
  expires: 2.weeks.from_now,
  httponly: true,
  secure: Rails.env.production?
}

# Signed cookies (tamper-proof)
cookies.signed[:user_id] = user.id

# Encrypted cookies (unreadable)
cookies.encrypted[:secret] = "sensitive data"

# Read cookies
cookies[:theme]           # => "dark"
cookies.signed[:user_id]  # => 1

# Delete cookies
cookies.delete(:theme)
```

## Resources

- [Action Controller Overview - Session](https://guides.rubyonrails.org/action_controller_overview.html#session) — Official guide to sessions in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*