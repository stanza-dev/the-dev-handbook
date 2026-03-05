---
source_course: "rails-foundations"
source_lesson: "rails-foundations-secure-passwords"
---

# Secure Password Authentication

## Introduction

Rails provides `has_secure_password` for handling passwords securely. It uses bcrypt to hash passwords so you never store plain text — even if your database is compromised, actual passwords remain protected.

## Key Concepts

- **`has_secure_password`**: A Rails method that adds password hashing, authentication, and validation to a model.
- **`password_digest`**: The database column that stores the bcrypt hash. You never store the plain text password.
- **bcrypt**: A one-way hashing algorithm designed for passwords, intentionally slow to prevent brute force attacks.
- **`authenticate`**: A method added by `has_secure_password` that verifies a plain text password against the stored hash.

## Real World Context

Every application that handles user passwords must hash them before storage. Major data breaches have exposed millions of passwords stored in plain text. `has_secure_password` makes secure password handling trivial in Rails, following industry best practices out of the box.

## Deep Dive

### Setting Up

```ruby
# 1. Add bcrypt gem
# Gemfile
gem 'bcrypt', '~> 3.1.7'
```

```bash
# 2. Generate user model with password_digest
bin/rails generate model User email:string password_digest:string
bin/rails db:migrate
```

```ruby
# 3. Add has_secure_password
class User < ApplicationRecord
  has_secure_password

  validates :email, presence: true,
                    uniqueness: { case_sensitive: false },
                    format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :password, length: { minimum: 8 },
                       if: -> { new_record? || password.present? }
  normalizes :email, with: -> email { email.strip.downcase }
end
```

### What has_secure_password Provides

```ruby
user = User.new(email: "alice@test.com", password: "secure123",
                password_confirmation: "secure123")
user.save  # Stores bcrypt hash in password_digest

user.authenticate("wrong")     # => false
user.authenticate("secure123") # => user object
```

### Registration Controller

```ruby
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to root_path, notice: "Account created!"
    else
      render :new, status: :unprocessable_entity
    end
  end

  private
  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation)
  end
end
```

### Login Controller

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by(email: params[:email])
    if user&.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to root_path, notice: "Welcome back!"
    else
      flash.now[:alert] = "Invalid email or password"
      render :new, status: :unprocessable_entity
    end
  end
end
```

## Common Pitfalls

- **Naming the column `password` instead of `password_digest`**: `has_secure_password` specifically looks for a `password_digest` column.
- **Logging passwords**: Ensure `password` is filtered from logs. Rails does this by default in `config/initializers/filter_parameter_logging.rb`.
- **No minimum length validation**: `has_secure_password` validates presence but not length. Always add a minimum length validation.

## Best Practices

- Always use `password_digest` as the column name.
- Add length validations for passwords (minimum 8 characters).
- Use the safe navigation operator (`user&.authenticate`) to handle nil users gracefully.

## Summary

- `has_secure_password` adds bcrypt password hashing to any model with a `password_digest` column.
- It provides `password`, `password_confirmation` virtual attributes and the `authenticate` method.
- `authenticate("password")` returns the user object on success and `false` on failure.
- Always add length validations and use the bcrypt gem.
- Never store plain text passwords; Rails makes this easy to avoid.

## Code Examples

**has_secure_password adds bcrypt hashing via the password_digest column. The authenticate method returns the user on success or false on failure.**

```ruby
class User < ApplicationRecord
  has_secure_password
  validates :email, presence: true, uniqueness: true
  validates :password, length: { minimum: 8 },
                       if: -> { new_record? || password.present? }
end

user = User.create(email: "a@b.com", password: "secure123")
user.authenticate("wrong")     # => false
user.authenticate("secure123") # => #<User id: 1, ...>
```


## Resources

- [has_secure_password API](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) — API documentation for secure password handling

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*