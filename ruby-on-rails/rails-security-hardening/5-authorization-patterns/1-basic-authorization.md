---
source_course: "rails-security-hardening"
source_lesson: "rails-security-basic-authorization"
---

# Basic Authorization

Authorization determines what authenticated users can do.

## Simple Role-Based Access

```ruby
class User < ApplicationRecord
  ROLES = %w[user moderator admin].freeze
  
  validates :role, inclusion: { in: ROLES }
  
  def admin?
    role == 'admin'
  end
  
  def moderator?
    role.in?(%w[moderator admin])
  end
end

class AdminController < ApplicationController
  before_action :require_admin
  
  private
  
  def require_admin
    unless current_user&.admin?
      redirect_to root_path, alert: 'Access denied'
    end
  end
end
```

## Resource-Based Authorization

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:edit, :update, :destroy]
  before_action :authorize_post_owner, only: [:edit, :update, :destroy]
  
  private
  
  def set_post
    @post = Post.find(params[:id])
  end
  
  def authorize_post_owner
    unless @post.user_id == current_user.id || current_user.admin?
      redirect_to posts_path, alert: 'You cannot modify this post'
    end
  end
end
```

## Authorization in Views

```erb
<%= link_to 'Edit', edit_post_path(@post) if can_edit?(@post) %>

<% if current_user&.admin? %>
  <%= link_to 'Admin Panel', admin_path %>
<% end %>
```

## Authorization Helper

```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def can_edit?(resource)
    return false unless current_user
    resource.user_id == current_user.id || current_user.admin?
  end
  
  def can_delete?(resource)
    can_edit?(resource)
  end
end
```

## Scoping Queries

```ruby
class PostsController < ApplicationController
  def index
    @posts = authorized_posts
  end
  
  private
  
  def authorized_posts
    if current_user&.admin?
      Post.all
    elsif current_user
      Post.where(published: true).or(Post.where(user: current_user))
    else
      Post.where(published: true)
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*