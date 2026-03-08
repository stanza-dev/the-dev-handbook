---
source_course: "rails-security-hardening"
source_lesson: "rails-security-strong-parameters"
---

# Strong Parameters

## Introduction
Mass assignment vulnerabilities occur when an attacker includes unexpected parameters in a request to set attributes they should not control. Strong parameters provide a whitelist approach to controlling which attributes can be set through request params.

## Key Concepts
- **Mass Assignment**: Setting multiple model attributes at once from a hash of parameters.
- **Strong Parameters**: A Rails mechanism that requires explicitly permitting which request parameters are allowed for mass assignment.
- **Unpermitted Parameters**: Parameters that are silently filtered out (or logged/raised depending on configuration).

## Real World Context
Without strong parameters, an attacker could add `admin: true` or `role: 'superadmin'` to a registration form and elevate their privileges. This vulnerability has affected major applications including GitHub (in 2012). Strong parameters are enabled by default in modern Rails.

## Deep Dive

### The Mass Assignment Problem

Without protection, any parameter can set any attribute:

```ruby
# DANGEROUS — without strong params
User.create(params[:user])
# Attacker could pass: { admin: true, role: 'superadmin' }
```

This creates a user with admin privileges because the raw params hash includes the attacker's additional fields.

### Using Strong Parameters

Explicitly permit only the expected attributes:

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

  private

  def user_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation)
  end
end
```

The `require` method ensures the `:user` key exists. The `permit` method whitelists specific attributes. Any additional parameters (like `admin` or `role`) are silently stripped.

### Nested Attributes

Permit nested attributes for associated records:

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

Nested attributes use an array to specify the permitted fields for the associated model.

### Conditional Permitting

Allow different attributes based on user role:

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

This pattern ensures that the `role` attribute can only be set by administrators.

## Common Pitfalls
1. **Using `permit!` to allow everything** — This completely bypasses strong parameter protection. Never use it with user-supplied data.
2. **Permitting hash params with `{}`** — Writing `filters: {}` permits any key-value pair inside `filters`, which can be dangerous.

## Best Practices
1. **Keep permit lists minimal** — Only permit attributes the form actually needs. Review permit lists when adding new form fields.
2. **Raise on unpermitted parameters in development** — Set `action_on_unpermitted_parameters = :raise` to catch mistakes early.

## Summary
- Strong parameters prevent mass assignment attacks by whitelisting allowed attributes.
- Use `require` to ensure the expected key exists and `permit` to whitelist attributes.
- Never use `permit!` with user-supplied data.
- Use conditional permitting to allow different attributes based on user role.
- Configure development to raise on unpermitted parameters for early detection.

## Code Examples

**Strong parameters with conditional permitting — the role attribute is only allowed for admin users**

```ruby
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    @user.save
  end

  private

  def user_params
    permitted = [:name, :email, :password, :password_confirmation]
    permitted << :role if current_user&.admin?  # Only admins can set role
    params.require(:user).permit(permitted)
  end
end
```


## Resources

- [Rails Strong Parameters Guide](https://guides.rubyonrails.org/action_controller_overview.html#strong-parameters) — Official Rails guide on strong parameters

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*