---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-advanced-migrations"
---

# Advanced Migration Techniques

Beyond basic table creation, migrations handle complex schema changes, data transformations, and database-specific features.

## Reversible Migrations

Rails automatically reverses most migrations. For complex cases, use `reversible`:

```ruby
class AddFullNameToUsers < ActiveRecord::Migration[8.0]
  def change
    add_column :users, :full_name, :string

    reversible do |dir|
      dir.up do
        User.find_each do |user|
          user.update_column(:full_name, "#{user.first_name} #{user.last_name}")
        end
      end

      dir.down do
        # No action needed - column will be dropped
      end
    end
  end
end
```

Or use separate `up` and `down` methods:

```ruby
class ChangeStatusToInteger < ActiveRecord::Migration[8.0]
  def up
    # Add new column
    add_column :orders, :status_code, :integer, default: 0

    # Migrate data
    execute <<-SQL
      UPDATE orders SET status_code = CASE status
        WHEN 'pending' THEN 0
        WHEN 'processing' THEN 1
        WHEN 'shipped' THEN 2
        WHEN 'delivered' THEN 3
        ELSE 0
      END
    SQL

    # Remove old column
    remove_column :orders, :status

    # Rename new column
    rename_column :orders, :status_code, :status
  end

  def down
    # Reverse the process
    add_column :orders, :status_text, :string

    execute <<-SQL
      UPDATE orders SET status_text = CASE status
        WHEN 0 THEN 'pending'
        WHEN 1 THEN 'processing'
        WHEN 2 THEN 'shipped'
        WHEN 3 THEN 'delivered'
        ELSE 'pending'
      END
    SQL

    remove_column :orders, :status
    rename_column :orders, :status_text, :status
  end
end
```

## Execute Raw SQL

```ruby
class AddDatabaseFunctions < ActiveRecord::Migration[8.0]
  def up
    execute <<-SQL
      CREATE OR REPLACE FUNCTION update_timestamp()
      RETURNS TRIGGER AS $$
      BEGIN
        NEW.updated_at = NOW();
        RETURN NEW;
      END;
      $$ LANGUAGE plpgsql;
    SQL
  end

  def down
    execute "DROP FUNCTION IF EXISTS update_timestamp()"
  end
end
```

## Check Constraints

```ruby
class AddConstraintsToProducts < ActiveRecord::Migration[8.0]
  def change
    # Ensure price is positive
    add_check_constraint :products, "price > 0", name: "products_price_positive"

    # Ensure quantity is non-negative
    add_check_constraint :products, "quantity >= 0", name: "products_quantity_non_negative"

    # Ensure valid status
    add_check_constraint :orders,
      "status IN ('pending', 'processing', 'shipped', 'delivered')",
      name: "orders_valid_status"
  end
end
```

## Index Options

```ruby
class AddIndexes < ActiveRecord::Migration[8.0]
  def change
    # Partial index (only index some rows)
    add_index :articles, :published_at, where: "published = true"

    # Unique index with condition
    add_index :users, :email, unique: true, where: "deleted_at IS NULL"

    # Expression index (PostgreSQL)
    add_index :users, "LOWER(email)", name: "index_users_on_lower_email"

    # Concurrent index (no lock, PostgreSQL)
    add_index :large_table, :column, algorithm: :concurrently
  end
end
```

## Bulk Changes

```ruby
class ModifyUsersTable < ActiveRecord::Migration[8.0]
  def change
    change_table :users, bulk: true do |t|
      t.string :phone
      t.string :address
      t.remove :legacy_field
      t.index :phone
    end
  end
end
```

## Resources

- [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) — Complete guide to Rails migrations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*