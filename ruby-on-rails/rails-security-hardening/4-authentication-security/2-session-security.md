---
source_course: "rails-security-hardening"
source_lesson: "rails-security-session-security"
---

# Session Security

Sessions are critical to security. Rails provides several options for secure session management.

## Session Configuration

```ruby
# config/initializers/session_store.rb
Rails.application.config.session_store :cookie_store,
  key: '_myapp_session',
  secure: Rails.env.production?,    # HTTPS only in production
  httponly: true,                   # No JavaScript access
  same_site: :lax                   # CSRF protection
```

## Session Fixation Prevention

Regenerate session ID after login to prevent fixation attacks:

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])
    
    if user&.authenticate(params[:password])
      # Regenerate session ID
      reset_session
      
      session[:user_id] = user.id
      redirect_to dashboard_path
    else
      flash.now[:alert] = 'Invalid credentials'
      render :new
    end
  end
  
  def destroy
    reset_session  # Clear entire session
    redirect_to root_path, notice: 'Logged out'
  end
end
```

## Session Expiration

```ruby
class ApplicationController < ActionController::Base
  before_action :check_session_expiry
  
  private
  
  def check_session_expiry
    if session[:expires_at] && session[:expires_at] < Time.current
      reset_session
      redirect_to login_path, alert: 'Session expired'
    end
  end
  
  def current_user
    return unless session[:user_id]
    
    @current_user ||= User.find_by(id: session[:user_id])
    
    # Extend session on activity
    session[:expires_at] = 30.minutes.from_now if @current_user
    
    @current_user
  end
end
```

## Remember Me Token

```ruby
class User < ApplicationRecord
  has_secure_token :remember_token
  
  def remember_me!
    update!(remember_token: SecureRandom.urlsafe_base64)
  end
  
  def forget_me!
    update!(remember_token: nil)
  end
end

class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])
    
    if user&.authenticate(params[:password])
      reset_session
      session[:user_id] = user.id
      
      if params[:remember_me]
        user.remember_me!
        cookies.encrypted[:remember_token] = {
          value: user.remember_token,
          expires: 2.weeks.from_now,
          httponly: true,
          secure: Rails.env.production?
        }
      end
      
      redirect_to dashboard_path
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*