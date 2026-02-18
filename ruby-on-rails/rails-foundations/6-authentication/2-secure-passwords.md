---
source_course: "rails-foundations"
source_lesson: "rails-foundations-secure-passwords"
---

# Secure Password Authentication

Rails provides `has_secure_password` for handling passwords securely. It uses bcrypt to hash passwords, so you never store plain text.

## Setting Up User Model

### 1. Add bcrypt gem

```ruby
# Gemfile
gem 'bcrypt', '~> 3.1.7'
```

Run `bundle install`.

### 2. Create User model with password_digest

```bash
bin/rails generate model User email:string password_digest:string
bin/rails db:migrate
```

### 3. Add has_secure_password

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password

  validates :email, presence: true,
                    uniqueness: { case_sensitive: false },
                    format: { with: URI::MailTo::EMAIL_REGEXP }

  validates :password, length: { minimum: 8 }, if: -> { new_record? || password.present? }

  # Normalize email before saving
  normalizes :email, with: -> email { email.strip.downcase }
end
```

## What has_secure_password Provides

```ruby
# Virtual attributes (not stored in DB)
user.password              # Set plain text password
user.password_confirmation # Optional confirmation

# Validation
user.password = "short"
user.valid?  # => false (if you add length validation)

# Authentication
user.authenticate("wrong")    # => false
user.authenticate("correct")  # => user object
```

## Registration Flow

### Routes

```ruby
# config/routes.rb
resources :users, only: [:new, :create]
get "signup", to: "users#new"
```

### Controller

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)

    if @user.save
      session[:user_id] = @user.id  # Log them in
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

### Registration Form

```erb
<!-- app/views/users/new.html.erb -->
<h1>Sign Up</h1>

<%= form_with model: @user, url: users_path do |f| %>
  <% if @user.errors.any? %>
    <div class="errors">
      <% @user.errors.full_messages.each do |msg| %>
        <p><%= msg %></p>
      <% end %>
    </div>
  <% end %>

  <div>
    <%= f.label :email %>
    <%= f.email_field :email, required: true %>
  </div>

  <div>
    <%= f.label :password %>
    <%= f.password_field :password, required: true %>
  </div>

  <div>
    <%= f.label :password_confirmation %>
    <%= f.password_field :password_confirmation %>
  </div>

  <%= f.submit "Create Account" %>
<% end %>
```

## Login Flow

### Routes

```ruby
get "login", to: "sessions#new"
post "login", to: "sessions#create"
delete "logout", to: "sessions#destroy"
```

### Sessions Controller

```ruby
class SessionsController < ApplicationController
  def new
  end

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

  def destroy
    session[:user_id] = nil
    redirect_to root_path, notice: "Logged out successfully"
  end
end
```

## Resources

- [has_secure_password API](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) — API documentation for secure password handling

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*