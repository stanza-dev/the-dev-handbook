---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-safe-migrations"
---

# Safe Database Migrations

## Introduction
Database migrations in production require extra care. Operations that seem harmless in development — like adding an index or renaming a column — can lock tables for minutes on large datasets, causing downtime.

## Key Concepts
- **Table Lock**: Some DDL operations (adding indexes, changing column types) lock the entire table, blocking all reads and writes.
- **Concurrent Index**: Adding an index with `algorithm: :concurrently` avoids locking the table, but requires `disable_ddl_transaction!`.
- **strong_migrations Gem**: Catches unsafe migration patterns and suggests safe alternatives before you deploy.

## Real World Context
Adding an index to a 50-million-row users table can lock it for 10+ minutes. During that time, every login, signup, and page load that touches users will hang. The `strong_migrations` gem prevents this by blocking unsafe operations and showing safe alternatives.

## Deep Dive

### The strong_migrations Gem

```ruby
# Gemfile
gem 'strong_migrations'
```

### Safe Index Creation

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def change
    add_index :users, :email, algorithm: :concurrently
  end
end
```

### Safe Column Addition (PostgreSQL 11+)

```ruby
# Safe in PostgreSQL 11+ — no table rewrite needed
class AddStatusToOrders < ActiveRecord::Migration[8.1]
  def change
    add_column :orders, :status, :string, default: 'pending', null: false
  end
end
```

### Safe Column Rename (Multi-Step)

```ruby
# Step 1: Add new column
class AddFullNameToUsers < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :full_name, :string
  end
end

# Step 2: Backfill data (in a background job)
# User.in_batches.update_all('full_name = name')

# Step 3: Update code to use new column

# Step 4: Remove old column in a later deploy
class RemoveNameFromUsers < ActiveRecord::Migration[8.1]
  def change
    safety_assured { remove_column :users, :name, :string }
  end
end
```

### Running Migrations via Kamal

```bash
kamal app exec 'bin/rails db:migrate'
kamal deploy
```

## Common Pitfalls
1. **Adding an index without `algorithm: :concurrently`** — On large tables, a standard `add_index` locks the table for the entire index build. Always use concurrent.
2. **Renaming a column in a single migration** — During a rolling deploy, old containers still reference the old column name. They'll crash immediately.

## Best Practices
1. **Install strong_migrations** — It catches unsafe operations before they reach production and suggests safe alternatives.
2. **Always test migrations on production-sized data** — A migration that takes 100ms on development data might take 10 minutes on production.

## Summary
- Use `algorithm: :concurrently` with `disable_ddl_transaction!` for indexes on large tables.
- Rename columns in multiple deploys: add new, backfill, update code, remove old.
- Install `strong_migrations` to catch unsafe operations automatically.
- Test migrations on production-sized data before deploying.

## Code Examples

**Adding an index concurrently — the table remains readable and writable during the entire index build**

```ruby
# Safe concurrent index — no table lock!
class AddIndexToOrdersUserId < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def change
    add_index :orders, :user_id, algorithm: :concurrently
  end
end

# disable_ddl_transaction! is required because
# concurrent indexes cannot run inside a transaction
```


## Resources

- [strong_migrations Gem](https://github.com/ankane/strong_migrations) — Gem that catches unsafe migrations and suggests safe alternatives

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*