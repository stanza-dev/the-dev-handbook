---
source_course: "rails-security-hardening"
source_lesson: "rails-security-strong-parameters"
---

# Strong Parameters

Strong parameters prevent attackers from setting attributes they shouldn't.

## The Mass Assignment Problem

Without protection, attackers can set any attribute:

```ruby
# DANGEROUS - without strong params
User.create(params[:user])
# Attacker could pass: { admin: true, role: 'superadmin' }
```

## Using Strong Parameters

```ruby
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to @user
    else
      render :new
    end
  end
  
  def update
    @user = User.find(params[:id])
    if @user.update(user_params)
      redirect_to @user
    else
      render :edit
    end
  end
  
  private
  
  def user_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation)
  end
end
```

## Nested Attributes

```ruby
def article_params
  params.require(:article).permit(
    :title,
    :body,
    :published,
    tags_attributes: [:id, :name, :_destroy],
    images_attributes: [:id, :url, :caption]
  )
end
```

## Conditional Permitting

```ruby
def user_params
  permitted = [:name, :email]
  
  # Only admins can set role
  permitted << :role if current_user.admin?
  
  # Only allow password on create or if explicitly changing
  if action_name == 'create' || params[:user][:password].present?
    permitted += [:password, :password_confirmation]
  end
  
  params.require(:user).permit(permitted)
end
```

## Array Parameters

```ruby
def filter_params
  params.permit(
    :search,
    category_ids: [],    # Array of scalars
    filters: {}          # Hash (be careful!)
  )
end
```

## Logging Unpermitted Parameters

```ruby
# config/environments/development.rb
config.action_controller.action_on_unpermitted_parameters = :raise

# config/environments/production.rb  
config.action_controller.action_on_unpermitted_parameters = :log
```

See [Strong Parameters Guide](https://guides.rubyonrails.org/action_controller_overview.html#strong-parameters).

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*