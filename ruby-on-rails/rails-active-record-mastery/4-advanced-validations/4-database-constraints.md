---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-database-constraints"
---

# Database-Level Constraints

## Introduction
Model validations run in Ruby and can be bypassed. Database constraints are the last line of defense, ensuring data integrity regardless of how data enters the database. A well-designed schema uses both model validations (for user-friendly error messages) and database constraints (for bulletproof integrity).

## Key Concepts
- **NOT NULL constraint**: Prevents a column from storing NULL values. The database equivalent of `validates :field, presence: true`.
- **UNIQUE constraint/index**: Ensures no two rows share the same value. The database equivalent of `validates :field, uniqueness: true`.
- **CHECK constraint**: Enforces arbitrary boolean expressions on column values (e.g., `price > 0`).
- **Foreign key constraint**: Ensures referential integrity between related tables. Prevents orphaned records.
- **Exclusion constraint** (PostgreSQL): Prevents overlapping ranges or conflicting values.

## Real World Context
Race conditions can bypass model-level uniqueness validations: two requests check for uniqueness simultaneously, both pass, and both insert. Only a database UNIQUE index catches this. Similarly, `update_column` and raw SQL bypass all model validations. If your business cannot tolerate negative balances or duplicate emails, database constraints are mandatory.

## Deep Dive

### NOT NULL Constraints

```ruby
class AddNotNullToUsers < ActiveRecord::Migration[8.1]
  def change
    change_column_null :users, :email, false
    change_column_null :users, :name, false
  end
end
```

This raises a database error if any code attempts to save a NULL email, even through `update_column` or raw SQL.

### UNIQUE Indexes

```ruby
class AddUniqueEmailIndex < ActiveRecord::Migration[8.1]
  def change
    add_index :users, :email, unique: true

    # Conditional unique index (only active users)
    add_index :users, :username, unique: true,
              where: "deleted_at IS NULL",
              name: "index_active_users_on_username"
  end
end
```

A conditional unique index (also called a partial index) is useful for soft-delete patterns where deleted records should not block new registrations.

### CHECK Constraints

```ruby
class AddCheckConstraints < ActiveRecord::Migration[8.1]
  def change
    add_check_constraint :products, "price > 0",
      name: "products_price_positive"

    add_check_constraint :products, "quantity >= 0",
      name: "products_quantity_non_negative"

    add_check_constraint :orders,
      "status IN ('pending', 'processing', 'shipped', 'delivered')",
      name: "orders_valid_status"
  end
end
```

CHECK constraints enforce domain rules at the database level. They are supported in PostgreSQL, MySQL 8.0+, and SQLite.

### Foreign Key Constraints

```ruby
class AddForeignKeys < ActiveRecord::Migration[8.1]
  def change
    add_foreign_key :orders, :users, on_delete: :cascade
    add_foreign_key :comments, :articles, on_delete: :nullify
    add_foreign_key :line_items, :orders, on_delete: :restrict
  end
end
```

The `on_delete` option controls what happens when the parent is deleted:
- `:cascade` -- deletes child records automatically
- `:nullify` -- sets the foreign key to NULL
- `:restrict` -- prevents deletion if children exist

### Exclusion Constraints (PostgreSQL)

```ruby
class AddExclusionConstraint < ActiveRecord::Migration[8.1]
  def up
    execute <<-SQL
      ALTER TABLE reservations
      ADD CONSTRAINT no_overlapping_reservations
      EXCLUDE USING gist (
        room_id WITH =,
        daterange(check_in, check_out) WITH &&
      );
    SQL
  end

  def down
    execute "ALTER TABLE reservations DROP CONSTRAINT no_overlapping_reservations"
  end
end
```

Exclusion constraints prevent overlapping date ranges, making them perfect for booking systems.

## Common Pitfalls
1. **Forgetting to handle database errors** -- When a constraint is violated, ActiveRecord raises `ActiveRecord::StatementInvalid`. Rescue it and present a user-friendly message.
2. **Duplicate validation logic** -- Having `validates :email, uniqueness: true` in the model AND a unique index is correct. The model validation gives a nice error message; the index prevents race conditions.
3. **Missing foreign keys** -- Rails does not add foreign key constraints by default. Always add them explicitly for referential integrity.

## Best Practices
1. **Layer both model validations and database constraints** -- Model validations for UX, database constraints for integrity.
2. **Name your constraints** -- Always use the `name:` option so error messages and migration rollbacks are clear.
3. **Use `on_delete: :cascade` carefully** -- Cascading deletes can remove more data than intended. Prefer `:restrict` and handle deletion explicitly.

## Summary
- NOT NULL, UNIQUE, CHECK, and foreign key constraints enforce data integrity at the database level.
- Model validations can be bypassed; database constraints cannot.
- Use both layers: model validations for user-friendly errors, constraints for bulletproof integrity.
- Exclusion constraints (PostgreSQL) prevent overlapping ranges.
- Always name constraints explicitly for maintainability.

## Code Examples

**Database-level constraints in a Rails migration -- NOT NULL, UNIQUE, CHECK, and foreign key constraints**

```ruby
class AddConstraints < ActiveRecord::Migration[8.1]
  def change
    # NOT NULL
    change_column_null :users, :email, false

    # UNIQUE index
    add_index :users, :email, unique: true

    # CHECK constraint
    add_check_constraint :accounts, "balance >= 0",
      name: "accounts_positive_balance"

    # Foreign key with cascade delete
    add_foreign_key :orders, :users, on_delete: :cascade
  end
end
```


## Resources

- [Active Record Migrations - Schema Statements](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html) — API reference for migration methods including constraints

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*