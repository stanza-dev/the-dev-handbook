---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-zero-downtime-migrations"
---

# Zero-Downtime Migrations

In production, you need to deploy without breaking your running application. This requires careful migration strategies.

## The Problem

Dangerous operations during deployment:

1. **Adding columns with defaults** (locks table in older databases)
2. **Removing columns** (code might still reference them)
3. **Renaming columns** (breaks running code)
4. **Adding NOT NULL** (existing NULLs cause failure)
5. **Large data migrations** (long-running transactions)

## Safe Patterns

### Adding Columns Safely

```ruby
# Step 1: Add column without default (deploy)
class AddStatusToOrders < ActiveRecord::Migration[8.0]
  def change
    add_column :orders, :status, :string
  end
end

# Step 2: Backfill data (run after deploy)
class BackfillOrderStatus < ActiveRecord::Migration[8.0]
  disable_ddl_transaction!  # Don't wrap in transaction

  def up
    Order.unscoped.in_batches do |batch|
      batch.update_all(status: "pending")
    end
  end
end

# Step 3: Add default and NOT NULL (deploy)
class AddDefaultToOrderStatus < ActiveRecord::Migration[8.0]
  def change
    change_column_default :orders, :status, "pending"
    change_column_null :orders, :status, false
  end
end
```

### Removing Columns Safely

```ruby
# Step 1: Stop using column in code (deploy)
# Update model to ignore column:
class User < ApplicationRecord
  self.ignored_columns += ["legacy_field"]
end

# Step 2: Remove column (next deploy)
class RemoveLegacyFieldFromUsers < ActiveRecord::Migration[8.0]
  def change
    remove_column :users, :legacy_field, :string
  end
end
```

### Renaming Columns Safely

```ruby
# DON'T: rename_column :users, :name, :full_name

# DO: Add new, copy data, update code, remove old

# Step 1: Add new column
class AddFullNameToUsers < ActiveRecord::Migration[8.0]
  def change
    add_column :users, :full_name, :string
  end
end

# Step 2: Copy data and sync writes
class SyncFullNameColumn < ActiveRecord::Migration[8.0]
  def up
    execute "UPDATE users SET full_name = name WHERE full_name IS NULL"
  end
end

# In model:
class User < ApplicationRecord
  before_save :sync_name_columns

  def sync_name_columns
    self.full_name = name if name_changed?
    self.name = full_name if full_name_changed?
  end
end

# Step 3: Update all code to use full_name
# Step 4: Remove old column
```

### Adding Indexes Safely (PostgreSQL)

```ruby
class AddIndexToLargeTable < ActiveRecord::Migration[8.0]
  disable_ddl_transaction!  # Required for CONCURRENTLY

  def change
    add_index :large_table, :column, algorithm: :concurrently
  end
end
```

## Strong Migrations Gem

Use the `strong_migrations` gem to catch dangerous migrations:

```ruby
# Gemfile
gem "strong_migrations"
```

It will warn you about:
- Adding columns with defaults
- Changing column types
- Renaming columns/tables
- Adding indexes without CONCURRENTLY
- And more...

## Batch Processing Large Data

```ruby
class BackfillLargeTable < ActiveRecord::Migration[8.0]
  disable_ddl_transaction!

  def up
    say_with_time "Backfilling records" do
      LargeModel.unscoped.in_batches(of: 1000) do |batch|
        batch.update_all(new_column: "default_value")
        sleep(0.1)  # Reduce database load
      end
    end
  end
end
```

## Resources

- [Strong Migrations Gem](https://github.com/ankane/strong_migrations) — Catch unsafe migrations in development

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*