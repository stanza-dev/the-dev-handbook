---
source_course: "rails-security-hardening"
source_lesson: "rails-security-policy-objects"
---

# Policy Objects

Policy objects encapsulate authorization logic in dedicated classes.

## Basic Policy Structure

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
    owner? || admin?
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

## Using Policies in Controllers

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]
  
  def show
    authorize! :show
    # ...
  end
  
  def edit
    authorize! :update
    # ...
  end
  
  def update
    authorize! :update
    # ...
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

## Application-Wide Policy Helper

```ruby
# app/controllers/application_controller.rb
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

## Using in Views

```erb
<% if policy_for(@post).update? %>
  <%= link_to 'Edit', edit_post_path(@post) %>
<% end %>

<% if policy_for(@post).destroy? %>
  <%= button_to 'Delete', @post, method: :delete %>
<% end %>
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*