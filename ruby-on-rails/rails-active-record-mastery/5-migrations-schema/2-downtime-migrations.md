---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-zero-downtime-migrations"
---

# Zero-Downtime Migrations

## Introduction
In production, you need to deploy schema changes without breaking your running application. Some migration operations lock tables, break running code, or cause errors for in-flight requests. Zero-downtime migration strategies let you evolve your schema safely.

## Key Concepts
- **Zero-downtime migration**: A schema change that does not cause errors or performance degradation for the running application.
- **`ignored_columns`**: Tells Active Record to pretend a column does not exist, preventing errors when removing columns.
- **Multi-step migration**: Spreading a dangerous change across multiple deployments to avoid breaking running code.
- **`strong_migrations` gem**: A tool that catches dangerous migrations in development before they reach production.

## Real World Context
Every growing Rails application eventually needs to remove columns, rename fields, change types, or add NOT NULL constraints. In a zero-downtime deployment environment (which is the standard for SaaS applications), you cannot simply run `remove_column` because the previous version of your code is still running and querying that column. Multi-step migration strategies are essential knowledge.

## Deep Dive

### Removing Columns Safely

The most common zero-downtime pattern:

```ruby
# Step 1: Stop using column in code (deploy first)
class User < ApplicationRecord
  self.ignored_columns += ["legacy_field"]
end

# Step 2: Remove column (deploy after Step 1 is live)
class RemoveLegacyFieldFromUsers < ActiveRecord::Migration[8.1]
  def change
    remove_column :users, :legacy_field, :string
  end
end
```

The `ignored_columns` setting makes Active Record skip the column when building SQL queries. Once the deploy with `ignored_columns` is live, the old code is no longer referencing the column, so the removal migration is safe.

### Adding Columns with Defaults Safely

```ruby
# Step 1: Add column without default
class AddStatusToOrders < ActiveRecord::Migration[8.1]
  def change
    add_column :orders, :status, :string
  end
end

# Step 2: Backfill data (run after deploy)
class BackfillOrderStatus < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def up
    Order.unscoped.in_batches do |batch|
      batch.update_all(status: "pending")
    end
  end
end

# Step 3: Add default and NOT NULL
class AddDefaultToOrderStatus < ActiveRecord::Migration[8.1]
  def change
    change_column_default :orders, :status, "pending"
    change_column_null :orders, :status, false
  end
end
```

In modern PostgreSQL, adding a column with a default value is fast and does not lock the table. But in older databases or for very large tables, this three-step approach is safer.

### Renaming Columns Safely

Never use `rename_column` directly. Instead:

```ruby
# Step 1: Add new column and sync writes
class AddFullNameToUsers < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :full_name, :string
  end
end

# In model: sync both columns during transition
class User < ApplicationRecord
  before_save :sync_name_columns

  private
  def sync_name_columns
    self.full_name = name if name_changed?
    self.name = full_name if full_name_changed?
  end
end

# Step 2: Backfill existing data
# Step 3: Switch all code to use full_name
# Step 4: Remove old column
```

### Adding Indexes Safely

```ruby
class AddIndexToLargeTable < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def change
    add_index :large_table, :column, algorithm: :concurrently
  end
end
```

### Strong Migrations Gem

```ruby
# Gemfile
gem "strong_migrations"
```

This gem catches dangerous operations and suggests safe alternatives. It will warn you about removing columns without `ignored_columns`, adding indexes without `CONCURRENTLY`, and more.

## Common Pitfalls
1. **Removing a column that running code still queries** -- The old deployment is still live during rollout. Always add `ignored_columns` first and deploy before removing the column.
2. **Renaming columns directly** -- `rename_column` breaks all running code that references the old name. Use the add-sync-migrate-remove pattern.
3. **Locking large tables** -- Adding a non-concurrent index on a table with millions of rows can lock it for minutes. Always use `algorithm: :concurrently` for large tables.

## Best Practices
1. **Install `strong_migrations`** -- It catches dangerous patterns automatically in development.
2. **Deploy migrations in multiple steps** -- One deployment to prepare, one to execute, one to clean up.
3. **Test migrations on production-sized data** -- A migration that runs in 1 second on your dev database might take 10 minutes on production.

## Summary
- Zero-downtime migrations spread dangerous changes across multiple deployments.
- Use `ignored_columns` before removing columns to prevent errors in running code.
- Use three-step add-backfill-constrain for adding columns with defaults on large tables.
- Never rename columns directly; use the add-sync-migrate-remove pattern.
- Use `algorithm: :concurrently` for indexes on large tables.
- Install the `strong_migrations` gem to catch dangerous patterns.

## Code Examples

**Two-step column removal -- first tell Active Record to ignore the column, then remove it after deployment**

```ruby
# Step 1: Deploy this model change first
class User < ApplicationRecord
  self.ignored_columns += ["legacy_field"]
end

# Step 2: After Step 1 is live, run this migration
class RemoveLegacyField < ActiveRecord::Migration[8.1]
  def change
    remove_column :users, :legacy_field, :string
  end
end
```


## Resources

- [Strong Migrations Gem](https://github.com/ankane/strong_migrations) — Catch unsafe migrations in development

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*