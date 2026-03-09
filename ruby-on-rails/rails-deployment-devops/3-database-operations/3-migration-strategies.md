---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-migration-strategies"
---

# Migration Deployment Strategies

## Introduction
During a zero-downtime rolling deploy, old and new code versions run simultaneously. Migrations must be compatible with both versions to avoid errors during the transition period.

## Key Concepts
- **Backwards-Compatible Migration**: A migration that works with both the old and new code versions running simultaneously.
- **Expand-Contract Pattern**: First expand the schema (add new columns/tables), deploy code that uses them, then contract (remove old columns) in a later deploy.
- **Backfill**: Populating new columns with data from existing columns, done in batches to avoid locking.

## Real World Context
During a Kamal rolling deploy to 3 servers, server 1 runs the new code while servers 2 and 3 still run old code. If you renamed a column, the old code on servers 2 and 3 will crash because the column they expect is gone.

## Deep Dive

### The Expand-Contract Pattern

**Deploy 1: Expand** — Add new column, deploy code that writes to both:

```ruby
class AddFullNameToUsers < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :full_name, :string
  end
end
```

```ruby
# Model writes to both columns during transition
class User < ApplicationRecord
  before_save :sync_full_name

  private

  def sync_full_name
    self.full_name = "#{first_name} #{last_name}"
  end
end
```

**Backfill** (run after deploy 1 is fully rolled out):

```ruby
# In a background job or rake task
User.in_batches(of: 1000).update_all(
  "full_name = CONCAT(first_name, ' ', last_name)"
)
```

**Deploy 2: Contract** — Remove old columns after all code uses the new one:

```ruby
class RemoveNameColumnsFromUsers < ActiveRecord::Migration[8.1]
  def change
    safety_assured do
      remove_column :users, :first_name, :string
      remove_column :users, :last_name, :string
    end
  end
end
```

### Adding a NOT NULL Constraint Safely

```ruby
# Deploy 1: Add column (nullable)
class AddStatusToOrders < ActiveRecord::Migration[8.1]
  def change
    add_column :orders, :status, :string
  end
end

# Backfill
Order.in_batches.update_all(status: 'pending')

# Deploy 2: Add constraint
class AddNotNullToOrdersStatus < ActiveRecord::Migration[8.1]
  def change
    change_column_null :orders, :status, false, 'pending'
  end
end
```

## Common Pitfalls
1. **Removing a column in the same deploy that stops using it** — During rolling deploy, old containers still reference the removed column and will crash.
2. **Backfilling in the migration itself** — Large backfills lock the table. Always backfill in a background job after the migration deploy.

## Best Practices
1. **Two-deploy minimum for schema changes** — Expand in deploy 1, contract in deploy 2. This ensures both code versions work during the transition.
2. **Use `in_batches` for backfills** — Process rows in chunks of 1000 to avoid locking and memory issues.

## Summary
- Use the expand-contract pattern for schema changes.
- Never remove a column in the same deploy that stops using it.
- Backfill data in background jobs, not in migrations.
- Plan at least two deploys for any column rename or removal.

## Code Examples

**Expand-contract pattern — add the new column first, backfill, then remove old columns in a separate deploy**

```ruby
# Deploy 1: Add new column (expand)
add_column :users, :full_name, :string

# Backfill after deploy 1 is fully rolled out
User.in_batches(of: 1000).update_all(
  "full_name = CONCAT(first_name, ' ', last_name)"
)

# Deploy 2: Remove old columns (contract)
safety_assured do
  remove_column :users, :first_name
  remove_column :users, :last_name
end
```


## Resources

- [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) — Official Rails guide on database migrations

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*