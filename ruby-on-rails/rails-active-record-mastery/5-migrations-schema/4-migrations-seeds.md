---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-data-migrations-seeds"
---

# Data Migrations and Seeds

## Introduction
Schema migrations change your database structure. Data migrations transform existing data to match new requirements. Seed files populate the database with initial or reference data. Understanding the difference and knowing the right strategies for each is critical for maintaining production databases.

## Key Concepts
- **Data migration**: A migration that modifies existing data (backfills, transforms, cleans up) rather than changing the schema.
- **Seed file**: A script (`db/seeds.rb`) that populates the database with initial data needed for the application to function.
- **Idempotent migration**: A migration that can be run multiple times without producing different results or errors.
- **`in_batches`**: Processes large datasets in chunks to avoid memory issues and long-running transactions.

## Real World Context
When you add a `full_name` column, you need to backfill it from `first_name` and `last_name`. When you change a status from string to enum, you need to transform existing values. When you split a table, you need to copy data. These are all data migrations. Getting them wrong can mean hours of downtime or lost data.

## Deep Dive

### Data Migration Strategies

#### Inline Data Migration (Small Tables)

For small tables (under 10,000 rows), you can transform data directly:

```ruby
class BackfillUserFullName < ActiveRecord::Migration[8.1]
  def up
    execute <<-SQL
      UPDATE users
      SET full_name = first_name || ' ' || last_name
      WHERE full_name IS NULL
    SQL
  end

  def down
    # Data migration -- no reverse needed
  end
end
```

Using raw SQL is fastest for simple transformations and avoids loading Active Record objects.

#### Batch Data Migration (Large Tables)

For large tables, process in batches to avoid locking and memory issues:

```ruby
class BackfillLargeTable < ActiveRecord::Migration[8.1]
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

The `sleep(0.1)` between batches prevents overwhelming the database on high-traffic systems. The `say_with_time` method logs progress in the migration output.

### Seed Files

The `db/seeds.rb` file populates initial data:

```ruby
# db/seeds.rb

# Idempotent: find_or_create_by prevents duplicates
Role.find_or_create_by!(name: "admin") do |role|
  role.description = "Full system access"
end

Role.find_or_create_by!(name: "editor") do |role|
  role.description = "Content management"
end

Role.find_or_create_by!(name: "viewer") do |role|
  role.description = "Read-only access"
end

# Environment-specific seeds
if Rails.env.development?
  10.times do |i|
    User.find_or_create_by!(email: "user#{i}@example.com") do |user|
      user.name = "Test User #{i}"
      user.role = Role.find_by!(name: "editor")
    end
  end
end
```

Key principle: seeds must be **idempotent**. Running `rails db:seed` twice should not create duplicate data or raise errors.

### Idempotent Migrations

```ruby
class BackfillSafely < ActiveRecord::Migration[8.1]
  def up
    # Only update rows that haven't been backfilled
    execute <<-SQL
      UPDATE orders
      SET normalized_status = LOWER(status)
      WHERE normalized_status IS NULL
    SQL
  end
end
```

The `WHERE normalized_status IS NULL` clause makes this safe to run multiple times.

### Separating Schema and Data Migrations

```ruby
# Migration 1: Schema change
class AddNormalizedEmailToUsers < ActiveRecord::Migration[8.1]
  def change
    add_column :users, :normalized_email, :string
    add_index :users, :normalized_email, unique: true
  end
end

# Migration 2: Data backfill (separate file)
class BackfillNormalizedEmail < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def up
    User.unscoped.in_batches(of: 5000) do |batch|
      batch.update_all("normalized_email = LOWER(email)")
    end
  end
end
```

Separating schema and data migrations makes each one simpler, reversible, and easier to debug.

## Common Pitfalls
1. **Using Active Record models in migrations** -- Model classes can change between when the migration was written and when it runs. Prefer raw SQL or use `reset_column_information` before accessing model attributes.
2. **Non-idempotent seeds** -- Using `create!` instead of `find_or_create_by!` causes errors on re-run. Always make seeds idempotent.
3. **Large single-transaction data migrations** -- Updating 10 million rows in one transaction can lock the table for minutes. Use `disable_ddl_transaction!` and `in_batches`.

## Best Practices
1. **Make data migrations idempotent** -- Use `WHERE column IS NULL` or `find_or_create_by` so they can be safely re-run.
2. **Separate schema and data migrations** -- One migration for the DDL change, another for the data backfill.
3. **Use raw SQL for performance** -- `batch.update_all("col = value")` is much faster than loading records into Ruby.

## Summary
- Data migrations transform existing data; schema migrations change structure.
- Use `in_batches` with `disable_ddl_transaction!` for large table backfills.
- Seed files must be idempotent using `find_or_create_by!`.
- Separate schema and data migrations for clarity and easier debugging.
- Prefer raw SQL over Active Record objects in migrations for performance and stability.

## Code Examples

**Idempotent seeds with find_or_create_by! and a batch data migration with in_batches for large tables**

```ruby
# Idempotent seed file
Role.find_or_create_by!(name: "admin") do |role|
  role.description = "Full system access"
end

# Batch data migration
class BackfillStatus < ActiveRecord::Migration[8.1]
  disable_ddl_transaction!

  def up
    Order.unscoped.where(status: nil).in_batches(of: 1000) do |batch|
      batch.update_all(status: "pending")
    end
  end
end
```


## Resources

- [Active Record Migrations - Running Migrations](https://guides.rubyonrails.org/active_record_migrations.html#running-migrations) — Guide to running and managing migrations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*