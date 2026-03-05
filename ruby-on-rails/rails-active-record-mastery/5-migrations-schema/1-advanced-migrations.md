---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-advanced-migrations"
---

# Advanced Migration Techniques

## Introduction
Beyond basic table creation, migrations handle complex schema changes, data transformations, and database-specific features. Mastering advanced migration techniques is essential for maintaining production databases without downtime or data loss.

## Key Concepts
- **`reversible` block**: Defines separate `up` and `down` logic within a `change` method for operations Rails cannot automatically reverse.
- **`execute`**: Runs raw SQL within a migration for database-specific features.
- **CHECK constraint**: A database-level rule that enforces a boolean expression on column values.
- **Partial index**: An index that only includes rows matching a condition, saving space and improving query speed.
- **`disable_ddl_transaction!`**: Opts out of wrapping the migration in a transaction, required for concurrent index creation.

## Real World Context
Production applications evolve constantly. You will rename columns, change types, add constraints, create database functions, and backfill data. Each operation has pitfalls that can lock tables, lose data, or break running code. Knowing the right migration technique for each scenario prevents production incidents.

## Deep Dive

### Reversible Migrations

Rails automatically reverses most operations (like `add_column`). For complex cases, use `reversible`:

```ruby
class AddFullNameToUsers < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :full_name, :string

    reversible do |dir|
      dir.up do
        User.find_each do |user|
          user.update_column(:full_name, "#{user.first_name} #{user.last_name}")
        end
      end
      dir.down do
        # No action needed -- column will be dropped
      end
    end
  end
end
```

The `reversible` block lets you define forward and backward behavior while keeping the main `change` method.

### Separate up and down Methods

For migrations that cannot use `change` at all:

```ruby
class ChangeStatusToInteger < ActiveRecord::Migration[8.1]
  def up
    add_column :orders, :status_code, :integer, default: 0
    execute <<-SQL
      UPDATE orders SET status_code = CASE status
        WHEN 'pending' THEN 0
        WHEN 'processing' THEN 1
        WHEN 'shipped' THEN 2
        ELSE 0
      END
    SQL
    remove_column :orders, :status
    rename_column :orders, :status_code, :status
  end

  def down
    add_column :orders, :status_text, :string
    execute <<-SQL
      UPDATE orders SET status_text = CASE status
        WHEN 0 THEN 'pending'
        WHEN 1 THEN 'processing'
        WHEN 2 THEN 'shipped'
        ELSE 'pending'
      END
    SQL
    remove_column :orders, :status
    rename_column :orders, :status_text, :status
  end
end
```

### Index Options

```ruby
class AddIndexes < ActiveRecord::Migration[8.1]
  def change
    # Partial index
    add_index :articles, :published_at, where: "published = true"

    # Expression index (PostgreSQL)
    add_index :users, "LOWER(email)", name: "index_users_on_lower_email"
  end
end

# Concurrent index (no table lock, PostgreSQL)
class AddConcurrentIndex < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def change
    add_index :large_table, :column, algorithm: :concurrently
  end
end
```

Concurrent indexes require `disable_ddl_transaction!` because they cannot run inside a transaction.

### Check Constraints

```ruby
class AddConstraints < ActiveRecord::Migration[8.1]
  def change
    add_check_constraint :products, "price > 0",
      name: "products_price_positive"
    add_check_constraint :orders,
      "status IN ('pending', 'processing', 'shipped', 'delivered')",
      name: "orders_valid_status"
  end
end
```

## Common Pitfalls
1. **Forgetting `disable_ddl_transaction!`** -- Concurrent index creation fails inside a transaction. Always add this directive for `algorithm: :concurrently`.
2. **Large data updates in transactions** -- Updating millions of rows in a single transaction can lock the table and exhaust memory. Use `in_batches` with `disable_ddl_transaction!`.
3. **Non-reversible migrations without `down`** -- If you use `execute` or `remove_column` in `change`, Rails may not know how to reverse it. Always define `down` or use `reversible`.

## Best Practices
1. **Name all constraints** -- Use the `name:` option so constraints are easy to identify in error messages and rollbacks.
2. **Use `in_batches` for data migrations** -- Process large tables in chunks to avoid locking and memory issues.
3. **Separate schema and data migrations** -- Keep DDL changes (column additions) and DML changes (data backfills) in separate migrations for clarity.

## Summary
- Use `reversible` or separate `up`/`down` methods for operations Rails cannot auto-reverse.
- `execute` runs raw SQL for database-specific features like functions and triggers.
- Partial indexes and expression indexes optimize queries on subsets of data.
- `disable_ddl_transaction!` is required for concurrent index creation.
- Always name constraints and separate schema changes from data backfills.

## Code Examples

**A reversible migration that adds a column and backfills data -- the down direction drops the column automatically**

```ruby
class AddFullNameWithBackfill < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :full_name, :string

    reversible do |dir|
      dir.up do
        User.in_batches.update_all("full_name = first_name || ' ' || last_name")
      end
    end
  end
end
```


## Resources

- [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) — Complete guide to Rails migrations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*