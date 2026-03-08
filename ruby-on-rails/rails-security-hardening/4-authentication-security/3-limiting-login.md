---
source_course: "rails-security-hardening"
source_lesson: "rails-security-rate-limiting-login"
---

# Rate Limiting Login

## Introduction
Without rate limiting, attackers can try thousands of passwords per minute in brute force attacks. Rails 8 includes built-in rate limiting, making it easy to protect login endpoints without external gems.

## Key Concepts
- **Rate Limiting**: Restricting the number of requests a client can make in a given time period.
- **Brute Force Attack**: Systematically trying every possible password until the correct one is found.
- **Credential Stuffing**: Using lists of stolen credentials from other breaches to attempt logins.
- **Account Lockout**: Temporarily disabling an account after too many failed login attempts.

## Real World Context
Credential stuffing attacks are automated and run at massive scale. Rate limiting is your first line of defense. Rails 8's built-in `rate_limit` method makes this trivial to implement.

## Deep Dive

### Rails 8 Built-in Rate Limiting

Rails 8 provides a `rate_limit` class method directly on controllers:

```ruby
class SessionsController < ApplicationController
  rate_limit to: 10, within: 3.minutes, only: :create

  def create
    if user = User.authenticate_by(
      email_address: params[:email_address],
      password: params[:password]
    )
      start_new_session_for(user)
      redirect_to root_path
    else
      redirect_to new_session_path, alert: 'Invalid credentials'
    end
  end
end
```

This limits login attempts to 10 per 3-minute window per IP address. When the limit is exceeded, Rails returns a `429 Too Many Requests` response automatically.

### Custom Rate Limit Responses

You can customize the response when limits are exceeded:

```ruby
class SessionsController < ApplicationController
  rate_limit to: 10, within: 3.minutes, only: :create,
    with: -> { redirect_to new_session_path, alert: 'Too many login attempts. Please wait.' }
end
```

The `with` parameter accepts a lambda that defines the response behavior.

### Rack::Attack for Complex Scenarios

For more advanced rate limiting (per-email throttling, IP blocking, geo-based rules), Rack::Attack remains an option:

```ruby
# Gemfile
gem 'rack-attack'

# config/initializers/rack_attack.rb
class Rack::Attack
  throttle('login/email', limit: 10, period: 1.minute) do |req|
    if req.path == '/session' && req.post?
      req.params.dig('email_address')&.downcase
    end
  end
end
```

Rack::Attack provides per-email throttling, which prevents an attacker from targeting a single account from multiple IP addresses.

### Account Lockout

For additional protection, implement account lockout after repeated failures:

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
    update!(failed_attempts: 0, locked_at: nil, unlock_token: nil)
  end
end
```

This locks the account after 5 failed attempts and sends an unlock email. The lock expires after 30 minutes.

## Common Pitfalls
1. **Only rate limiting by IP** — Attackers use botnets with thousands of IPs. Add per-email throttling to protect individual accounts.
2. **Revealing whether an account exists** — Messages like "No user found with that email" help attackers enumerate valid accounts. Use a generic message.

## Best Practices
1. **Use Rails 8's built-in `rate_limit` for simple cases** — It requires no gems and covers most login protection needs.
2. **Combine rate limiting with account lockout** — Rate limiting slows attacks; lockout stops them after repeated failures.

## Summary
- Rails 8 provides built-in `rate_limit` for controller-level throttling.
- The built-in method limits requests per IP with automatic 429 responses.
- Rack::Attack is available for complex scenarios like per-email throttling.
- Account lockout adds a second layer by disabling accounts after failed attempts.
- Use generic error messages to prevent account enumeration.

## Code Examples

**Rails 8 built-in rate limiting — no gems needed, limits login attempts per IP address**

```ruby
class SessionsController < ApplicationController
  # Rails 8 built-in rate limiting
  rate_limit to: 10, within: 3.minutes, only: :create

  def create
    if user = User.authenticate_by(
      email_address: params[:email_address],
      password: params[:password]
    )
      start_new_session_for(user)
      redirect_to root_path
    else
      redirect_to new_session_path, alert: 'Invalid credentials'
    end
  end
end
# Exceeding 10 requests in 3 minutes returns 429 Too Many Requests
```


## Resources

- [Rails Rate Limiting](https://api.rubyonrails.org/classes/ActionController/RateLimiting/ClassMethods.html) — API reference for Rails 8 built-in rate limiting

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*