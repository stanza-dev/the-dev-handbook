---
source_course: "rails-security-hardening"
source_lesson: "rails-security-policy-objects"
---

# Policy Objects

## Introduction
As authorization rules grow complex, scattering them across controllers becomes unmaintainable. Policy objects encapsulate authorization logic in dedicated classes, making rules testable, reusable, and easy to audit.

## Key Concepts
- **Policy Object**: A plain Ruby class that encapsulates all authorization rules for a specific resource type.
- **Authorization Methods**: Boolean methods on the policy (like `show?`, `update?`, `destroy?`) that return whether the action is permitted.
- **Policy Helper**: A controller method that instantiates the correct policy and makes it available in views.

## Real World Context
Gems like Pundit formalize the policy object pattern, but you can implement it with plain Ruby. Policy objects make authorization auditable — a security reviewer can read one file to understand all access rules for a resource.

## Deep Dive

### Basic Policy Structure

Create a policy class that takes a user and a resource:

```ruby
# app/policies/post_policy.rb
class PostPolicy
  attr_reader :user, :post

  def initialize(user, post)
    @user = user
    @post = post
  end

  def show?
    post.published? || owner? || admin?
  end

  def create?
    user.present?
  end

  def update?
    owner? || admin?
  end

  def destroy?
    owner?
  end

  def publish?
    owner? && post.draft?
  end

  private

  def owner?
    user && post.user_id == user.id
  end

  def admin?
    user&.admin?
  end
end
```

Each method answers a specific authorization question. Notice that `destroy?` only allows the owner — even admins cannot delete other users' posts in this policy.

### Using Policies in Controllers

Integrate the policy into controller actions:

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def show
    authorize! :show
  end

  def update
    authorize! :update
    if @post.update(post_params)
      redirect_to @post
    else
      render :edit
    end
  end

  def destroy
    authorize! :destroy
    @post.destroy
    redirect_to posts_path
  end

  private

  def set_post
    @post = Post.find(params[:id])
  end

  def policy
    @policy ||= PostPolicy.new(current_user, @post)
  end

  def authorize!(action)
    unless policy.send("#{action}?")
      raise NotAuthorizedError, "Not authorized to #{action} this post"
    end
  end
end
```

The `authorize!` method checks the policy and raises an error if the action is not permitted.

### Application-Wide Error Handling

Handle authorization failures globally:

```ruby
class ApplicationController < ActionController::Base
  class NotAuthorizedError < StandardError; end

  rescue_from NotAuthorizedError do |e|
    redirect_to root_path, alert: 'You are not authorized to perform this action.'
  end

  def policy_for(record)
    "#{record.class}Policy".constantize.new(current_user, record)
  end
  helper_method :policy_for
end
```

The `policy_for` helper is available in views for conditionally showing UI elements.

### Using in Views

Conditionally show UI elements based on authorization:

```erb
<% if policy_for(@post).update? %>
  <%= link_to 'Edit', edit_post_path(@post) %>
<% end %>

<% if policy_for(@post).destroy? %>
  <%= button_to 'Delete', @post, method: :delete %>
<% end %>
```

This ensures buttons only appear for authorized users, but remember to also enforce authorization in the controller.

## Common Pitfalls
1. **Only using policies in views** — Hiding UI elements without controller enforcement is just obscurity, not security. An attacker can call the endpoint directly.
2. **Forgetting to test policies** — Policy objects are easy to unit test. Write tests for every permission method.

## Best Practices
1. **Keep policies focused** — One policy per model. If a policy gets too complex, it may indicate the model has too many responsibilities.
2. **Make destroy the most restrictive** — Deletion is irreversible. Consider making it owner-only even when update allows admins.

## Summary
- Policy objects encapsulate authorization rules in dedicated, testable classes.
- Each policy method (`show?`, `update?`, `destroy?`) answers a specific authorization question.
- Use `policy_for` in views to conditionally render UI elements.
- Always enforce policies in controllers, not just in views.
- Keep deletion as the most restrictive permission.

## Code Examples

**A policy object with distinct rules — update allows owner or admin, but destroy is restricted to owner only**

```ruby
# app/policies/post_policy.rb
class PostPolicy
  attr_reader :user, :post

  def initialize(user, post)
    @user = user
    @post = post
  end

  def update?
    owner? || admin?
  end

  def destroy?
    owner?  # Only owner can delete — most destructive action
  end

  private

  def owner?
    user && post.user_id == user.id
  end

  def admin?
    user&.admin?
  end
end
```


## Resources

- [Pundit Gem](https://github.com/varvet/pundit) — Popular authorization gem that formalizes the policy object pattern

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*