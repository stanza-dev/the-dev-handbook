---
source_course: "rails-security-hardening"
source_lesson: "rails-security-basic-authorization"
---

# Basic Authorization

## Introduction
Authentication answers "who are you?" — authorization answers "what can you do?" Without proper authorization, any authenticated user could access admin panels, modify other users' data, or delete resources they do not own.

## Key Concepts
- **Authorization**: The process of determining whether an authenticated user has permission to perform a specific action.
- **Role-Based Access Control (RBAC)**: Assigning permissions based on user roles (admin, moderator, user).
- **Resource-Based Authorization**: Checking permissions based on the relationship between the user and the specific resource.
- **Query Scoping**: Filtering database queries so users only see records they are authorized to access.

## Real World Context
Every multi-user application needs authorization. An e-commerce site must ensure customers can only view their own orders. A CMS must restrict publishing to editors. A SaaS platform must isolate tenant data. Authorization bugs are among the most common security vulnerabilities.

## Deep Dive

### Simple Role-Based Access

Define roles on the User model and check them in controllers:

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

The `moderator?` method includes admins because admins should have all moderator permissions.

### Resource-Based Authorization

Check ownership before allowing modifications:

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

This ensures that only the post owner or an admin can edit or delete a post.

### Scoping Queries

Filter data at the query level so users never see unauthorized records:

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

Query scoping prevents data leaks by ensuring the database only returns authorized records.

## Common Pitfalls
1. **Only checking authorization in views** — Hiding a button does not prevent a direct API call. Always enforce authorization in controllers.
2. **Forgetting to scope index queries** — Listing all records without filtering is a common source of data leaks.

## Best Practices
1. **Authorize at the controller level** — Use `before_action` callbacks to enforce authorization before any action logic runs.
2. **Scope all queries** — Always filter queries based on the current user's permissions to prevent unauthorized data access.

## Summary
- Authorization controls what authenticated users can do.
- Role-based access assigns permissions by user role (admin, moderator, user).
- Resource-based authorization checks ownership of specific records.
- Query scoping ensures users only see data they are authorized to access.
- Always enforce authorization in controllers, never only in views.

## Code Examples

**Resource-based authorization — only the post owner or an admin can edit or delete a post**

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:edit, :update, :destroy]
  before_action :authorize_post_owner, only: [:edit, :update, :destroy]

  private

  def authorize_post_owner
    unless @post.user_id == current_user.id || current_user.admin?
      redirect_to posts_path, alert: 'You cannot modify this post'
    end
  end
end
```


## Resources

- [Rails Security Guide — Authorization](https://guides.rubyonrails.org/security.html#authorization) — Official Rails guide on implementing authorization

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*