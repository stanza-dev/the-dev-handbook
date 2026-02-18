---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-safe-migrations"
---

# Safe Migrations

Database migrations in production require extra care to avoid downtime.

## Dangerous Operations

These operations can lock tables and cause downtime:

- Adding indexes on large tables
- Adding columns with default values
- Renaming columns/tables
- Changing column types

## Strong Migrations Gem

```ruby
# Gemfile
gem 'strong_migrations'
```

Detects unsafe migrations:

```ruby
class AddIndexToUsers < ActiveRecord::Migration[7.1]
  def change
    # This will raise an error!
    add_index :users, :email
  end
end
```

## Safe Migration Patterns

### Adding Indexes Concurrently

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration[7.1]
  disable_ddl_transaction!
  
  def change
    add_index :users, :email, algorithm: :concurrently
  end
end
```

### Adding Columns with Defaults

```ruby
# In PostgreSQL 11+, this is safe:
class AddStatusToOrders < ActiveRecord::Migration[7.1]
  def change
    add_column :orders, :status, :string, default: 'pending', null: false
  end
end

# For older versions, do it in steps:
class AddStatusToOrdersSafely < ActiveRecord::Migration[7.1]
  def up
    add_column :orders, :status, :string
    change_column_default :orders, :status, 'pending'
    
    # Backfill in batches
    Order.in_batches.update_all(status: 'pending')
    
    change_column_null :orders, :status, false
  end
end
```

### Renaming Columns

```ruby
# Step 1: Add new column
class AddNewNameColumn < ActiveRecord::Migration[7.1]
  def change
    add_column :users, :full_name, :string
  end
end

# Step 2: Backfill data (in a job)
User.in_batches.update_all('full_name = name')

# Step 3: Update code to use new column

# Step 4: Remove old column
class RemoveOldNameColumn < ActiveRecord::Migration[7.1]
  def change
    safety_assured { remove_column :users, :name }
  end
end
```

## Running Migrations

```bash
# Run migrations
kamal app exec --reuse 'bin/rails db:migrate'

# Or via SSH
ssh deploy@server 'cd /app && bin/rails db:migrate'

# Check migration status
bin/rails db:migrate:status
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*