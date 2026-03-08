---
source_course: "rails-security-hardening"
source_lesson: "rails-security-secure-password-storage"
---

# Secure Password Storage

## Introduction
Password storage is one of the most critical security decisions in any application. Storing passwords in plain text or using weak hashing algorithms has led to devastating breaches. Rails provides `has_secure_password` for secure password handling using bcrypt.

## Key Concepts
- **has_secure_password**: A Rails macro that adds bcrypt-based password hashing to a model.
- **bcrypt**: A password hashing algorithm designed to be slow, making brute-force attacks impractical.
- **Password Digest**: The hashed output stored in the database — the original password cannot be recovered from it.
- **Timing Attack**: An attack that measures response time differences to deduce secret values.

## Real World Context
Every data breach involving plain-text or weakly hashed passwords makes headlines. Using bcrypt through `has_secure_password` is the Rails standard. Rails 8 also provides a built-in authentication generator that sets this up automatically.

## Deep Dive

### Setting Up Secure Passwords

Rails 8 includes a built-in authentication generator that creates the full authentication scaffold:

```bash
bin/rails generate authentication
```

This generates a `User` model with `has_secure_password`, a `SessionsController`, an `Authentication` concern, and a `Current` model. For manual setup:

```ruby
class User < ApplicationRecord
  has_secure_password

  validates :password, length: { minimum: 8 }, allow_nil: true
end
```

This requires a `password_digest` column in the database:

```ruby
class AddPasswordDigestToUsers < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :password_digest, :string, null: false
  end
end
```

The migration uses `ActiveRecord::Migration[8.1]` to target the current Rails version.

### How It Works

`has_secure_password` provides password hashing and authentication methods:

```ruby
user = User.new(email: 'user@example.com')
user.password = 'secure_password_123'  # Hashes automatically with bcrypt
user.password_confirmation = 'secure_password_123'
user.save

# Later, authenticate:
user.authenticate('wrong_password')        # => false
user.authenticate('secure_password_123')   # => user object
```

The `password=` setter automatically hashes the password. The `authenticate` method compares a candidate password against the stored digest using bcrypt's built-in comparison.

### Password Reset Tokens with generates_token_for

Rails provides `generates_token_for` (introduced in Rails 7.1) for secure, expiring tokens:

```ruby
class User < ApplicationRecord
  has_secure_password

  generates_token_for :password_reset, expires_in: 15.minutes do
    password_salt&.last(10)
  end
end

# Generate a reset token
token = user.generate_token_for(:password_reset)

# Find user by token (returns nil if expired or password changed)
user = User.find_by_token_for(:password_reset, token)
```

The block includes `password_salt` so the token is automatically invalidated when the password changes — a user cannot reuse an old reset link.

### Preventing Timing Attacks

Use secure comparison for token validation:

```ruby
# WRONG — vulnerable to timing attack
if params[:token] == user.reset_token
  # ...
end

# RIGHT — constant-time comparison
if ActiveSupport::SecurityUtils.secure_compare(
     params[:token],
     user.reset_token
   )
  # ...
end
```

Constant-time comparison takes the same amount of time regardless of how many characters match, preventing attackers from guessing tokens one character at a time.

## Common Pitfalls
1. **Storing passwords in plain text or with MD5/SHA1** — These are fast hashing algorithms designed for data integrity, not password security. Always use bcrypt.
2. **Not setting a minimum password length** — Without length validation, users can set single-character passwords. Enforce at least 8 characters.

## Best Practices
1. **Use Rails 8's authentication generator** — `bin/rails generate authentication` sets up the entire authentication flow with best practices built in.
2. **Use `generates_token_for` for reset tokens** — It provides built-in expiration and automatic invalidation on password change.

## Summary
- `has_secure_password` provides bcrypt-based password hashing in Rails.
- The `password_digest` column stores the hashed password.
- Rails 8 includes `bin/rails generate authentication` for full auth scaffolding.
- `generates_token_for` creates secure, expiring tokens for password resets.
- Always use constant-time comparison for sensitive token validation.

## Code Examples

**Secure password setup with bcrypt and Rails 8's generates_token_for for password reset tokens**

```ruby
class User < ApplicationRecord
  has_secure_password
  validates :password, length: { minimum: 8 }, allow_nil: true

  # Rails 8: secure, expiring password reset tokens
  generates_token_for :password_reset, expires_in: 15.minutes do
    password_salt&.last(10)
  end
end

# Generate reset token
token = user.generate_token_for(:password_reset)

# Verify reset token (nil if expired or password changed)
user = User.find_by_token_for(:password_reset, token)
```


## Resources

- [has_secure_password API Reference](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) — Official API reference for Rails secure password functionality
- [Rails Authentication Generator Guide](https://guides.rubyonrails.org/security.html#user-management) — Official guide on Rails built-in authentication

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*