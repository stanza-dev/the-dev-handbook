---
source_course: "rails-security-hardening"
source_lesson: "rails-security-secure-password-storage"
---

# Secure Password Storage

Never store passwords in plain text. Rails provides `has_secure_password` for secure password handling.

## Setting Up Secure Passwords

```ruby
class User < ApplicationRecord
  has_secure_password
  
  validates :password, length: { minimum: 8 }, allow_nil: true
end
```

This requires a `password_digest` column:

```ruby
class AddPasswordDigestToUsers < ActiveRecord::Migration[7.1]
  def change
    add_column :users, :password_digest, :string, null: false
  end
end
```

## How It Works

`has_secure_password` provides:

```ruby
user = User.new(email: 'user@example.com')
user.password = 'secure_password_123'  # Hashes automatically
user.password_confirmation = 'secure_password_123'
user.save

# Later, authenticate:
user.authenticate('wrong_password')    # => false
user.authenticate('secure_password_123')  # => user object
```

## Password Validation

```ruby
class User < ApplicationRecord
  has_secure_password
  
  # Minimum length
  validates :password, length: { minimum: 12 }, on: :create
  
  # Custom validation for complexity
  validate :password_complexity
  
  private
  
  def password_complexity
    return if password.blank?
    
    unless password.match?(/[a-z]/) && password.match?(/[A-Z]/)
      errors.add :password, 'must include uppercase and lowercase letters'
    end
    
    unless password.match?(/\d/)
      errors.add :password, 'must include a number'
    end
  end
end
```

## Preventing Timing Attacks

Use secure comparison for tokens:

```ruby
# WRONG - vulnerable to timing attack
if params[:token] == user.reset_token
  # ...
end

# RIGHT - constant-time comparison
if ActiveSupport::SecurityUtils.secure_compare(
     params[:token],
     user.reset_token
   )
  # ...
end
```

See [has_secure_password docs](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html).

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*