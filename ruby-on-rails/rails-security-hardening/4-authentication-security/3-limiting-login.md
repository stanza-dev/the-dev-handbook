---
source_course: "rails-security-hardening"
source_lesson: "rails-security-rate-limiting-login"
---

# Rate Limiting Login

Rate limiting prevents attackers from trying thousands of passwords.

## Using Rack::Attack

Add to your Gemfile:
```ruby
gem 'rack-attack'
```

Configure rate limiting:

```ruby
# config/initializers/rack_attack.rb
class Rack::Attack
  ### Throttle login attempts by IP ###
  throttle('login/ip', limit: 5, period: 20.seconds) do |req|
    req.ip if req.path == '/login' && req.post?
  end
  
  ### Throttle login attempts by email ###
  throttle('login/email', limit: 10, period: 1.minute) do |req|
    if req.path == '/login' && req.post?
      req.params['email'].presence
    end
  end
  
  ### Block suspicious requests ###
  blocklist('block bad IPs') do |req|
    BannedIp.exists?(ip: req.ip)
  end
  
  ### Custom response ###
  self.throttled_responder = lambda do |env|
    retry_after = (env['rack.attack.match_data'] || {})[:period]
    [
      429,
      { 'Content-Type' => 'application/json', 'Retry-After' => retry_after.to_s },
      [{ error: 'Rate limit exceeded. Try again later.' }.to_json]
    ]
  end
end
```

## Account Lockout

```ruby
class User < ApplicationRecord
  def increment_failed_attempts!
    increment!(:failed_attempts)
    lock_account! if failed_attempts >= 5
  end
  
  def lock_account!
    update!(
      locked_at: Time.current,
      unlock_token: SecureRandom.urlsafe_base64
    )
    UserMailer.account_locked(self).deliver_later
  end
  
  def locked?
    locked_at.present? && locked_at > 30.minutes.ago
  end
  
  def unlock!
    update!(
      failed_attempts: 0,
      locked_at: nil,
      unlock_token: nil
    )
  end
end

class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])
    
    if user&.locked?
      flash[:alert] = 'Account locked. Check email to unlock.'
      render :new
      return
    end
    
    if user&.authenticate(params[:password])
      user.update!(failed_attempts: 0)
      sign_in(user)
    else
      user&.increment_failed_attempts!
      flash[:alert] = 'Invalid credentials'
      render :new
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*