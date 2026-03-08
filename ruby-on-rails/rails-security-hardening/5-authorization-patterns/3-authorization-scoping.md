---
source_course: "rails-security-hardening"
source_lesson: "rails-security-authorization-scoping"
---

# Authorization Scoping

## Introduction
While policy objects control access to individual records, query scoping controls which records appear in lists and searches. Without proper scoping, users can see data belonging to other users or tenants.

## Key Concepts
- **Default Scope Danger**: Using `default_scope` for authorization is fragile because it can be bypassed with `unscoped`.
- **Controller Scoping**: Applying authorization filters in controller methods to restrict query results.
- **Multi-Tenancy**: Isolating data between different organizations or accounts that share the same database.

## Real World Context
Data leaks through unscoped queries are one of the most common authorization bugs. A user viewing `/invoices` should only see their own invoices, not every invoice in the system. SaaS applications must isolate tenant data to prevent cross-tenant access.

## Deep Dive

### Controller-Level Scoping

Scope queries in the controller based on the current user:

```ruby
class InvoicesController < ApplicationController
  def index
    @invoices = current_user.invoices.order(created_at: :desc)
  end

  def show
    @invoice = current_user.invoices.find(params[:id])
  end
end
```

By querying through `current_user.invoices`, Rails automatically adds a `WHERE user_id = ?` condition. If a user tries to access another user's invoice by ID, `find` raises `ActiveRecord::RecordNotFound`.

### Scope Methods for Roles

Create scope methods that respect user roles:

```ruby
class PostsController < ApplicationController
  def index
    @posts = visible_posts.page(params[:page])
  end

  private

  def visible_posts
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

Admins see everything, logged-in users see published posts plus their own drafts, and anonymous users only see published posts.

### Multi-Tenant Scoping

For SaaS applications, scope all queries to the current organization:

```ruby
module TenantScoping
  extend ActiveSupport::Concern

  included do
    before_action :set_current_tenant
    around_action :scope_to_tenant
  end

  private

  def set_current_tenant
    Current.organization = current_user&.organization
  end

  def scope_to_tenant
    if Current.organization
      Project.where(organization: Current.organization).scoping do
        yield
      end
    else
      yield
    end
  end
end
```

The `scoping` block ensures all Project queries within the request are automatically filtered to the current organization.

## Common Pitfalls
1. **Using `Post.find(params[:id])` without scoping** — This allows any user to access any record by guessing IDs. Always scope through the current user or organization.
2. **Relying on `default_scope` for security** — `default_scope` can be bypassed with `unscoped`, making it unreliable for authorization.

## Best Practices
1. **Always scope through associations** — Use `current_user.posts.find(id)` instead of `Post.find(id)` when users should only access their own resources.
2. **Use `Current` attributes for tenant context** — Rails' `CurrentAttributes` provides a clean way to pass tenant context through the request.

## Summary
- Query scoping ensures users only see records they are authorized to access.
- Always scope `find` calls through associations to prevent unauthorized access.
- Role-based scoping shows different data sets based on user permissions.
- Multi-tenant applications must scope all queries to the current organization.
- Never rely on `default_scope` for security — use explicit controller scoping.

## Code Examples

**Scoped vs unscoped record access — always query through associations to prevent unauthorized access**

```ruby
class InvoicesController < ApplicationController
  # SAFE — scoped through current_user association
  def show
    @invoice = current_user.invoices.find(params[:id])
    # Raises RecordNotFound if invoice belongs to another user
  end

  # DANGEROUS — unscoped find allows access to any invoice
  # def show
  #   @invoice = Invoice.find(params[:id])
  # end
end
```


## Resources

- [Active Record Scoping](https://guides.rubyonrails.org/active_record_querying.html#scopes) — Official Rails guide on query scoping techniques

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*