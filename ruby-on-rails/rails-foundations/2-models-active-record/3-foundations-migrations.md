---
source_course: "rails-foundations"
source_lesson: "rails-foundations-migrations"
---

# Database Migrations

## Introduction

Migrations are Ruby classes that let you modify your database schema over time. They provide version control for your database, so your entire team stays in sync and changes can be rolled back safely.

## Key Concepts

- **Migration**: A Ruby class that defines a set of reversible database schema changes.
- **Schema file** (`db/schema.rb`): An auto-generated snapshot of your current database structure.
- **Column types**: Rails supports `:string`, `:text`, `:integer`, `:boolean`, `:datetime`, `:decimal`, `:json`, and more.
- **Timestamps**: `t.timestamps` adds `created_at` and `updated_at` columns automatically.

## Real World Context

In a team environment, you cannot just edit the database directly. Migrations let each developer apply the same schema changes in order. They are committed to version control, so deploying to production is just running `bin/rails db:migrate`.

## Deep Dive

### Creating Migrations

```bash
# Generate a migration
bin/rails generate migration AddPublishedToArticles published:boolean
```

```ruby
class AddPublishedToArticles < ActiveRecord::Migration[8.1]
  def change
    add_column :articles, :published, :boolean, default: false
  end
end
```

### Common Migration Methods

```ruby
# Create a table
create_table :products do |t|
  t.string :name, null: false
  t.decimal :price, precision: 10, scale: 2
  t.boolean :active, default: true
  t.references :category, foreign_key: true
  t.timestamps
end

# Modify columns
add_column :articles, :published_at, :datetime
remove_column :articles, :author_name, :string
rename_column :articles, :title, :headline
add_index :users, :email, unique: true
```

### Running Migrations

```bash
bin/rails db:migrate          # Run pending migrations
bin/rails db:rollback         # Undo last migration
bin/rails db:rollback STEP=3  # Undo last 3
bin/rails db:migrate:status   # Check status
```

### The Schema File

```ruby
ActiveRecord::Schema[8.1].define(version: 2024_01_01_120000) do
  create_table "articles", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.boolean "published", default: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
end
```

## Common Pitfalls

- **Editing `db/schema.rb` directly**: This file is auto-generated. Always use migrations to change the schema.
- **Non-reversible migrations**: Use `change` method instead of separate `up`/`down` methods when possible, so Rails can auto-reverse.
- **Forgetting indexes on foreign keys**: Always add indexes on columns used in `WHERE` clauses and foreign keys.

## Best Practices

- Run `bin/rails db:migrate` immediately after creating a migration.
- Never edit a migration that has already been run in production; create a new one instead.
- Use `null: false` and `default:` values to enforce data integrity at the database level.

## Summary

- Migrations version-control your database schema using Ruby instead of raw SQL.
- Use generators to create migrations: `bin/rails generate migration Name`.
- Common methods: `create_table`, `add_column`, `remove_column`, `add_index`.
- Run with `bin/rails db:migrate`, roll back with `bin/rails db:rollback`.
- The schema file (`db/schema.rb`) is auto-generated and should not be edited manually.

## Code Examples

**A migration that creates a products table with various column types, a foreign key reference, and automatic timestamps.**

```ruby
class CreateProducts < ActiveRecord::Migration[8.1]
  def change
    create_table :products do |t|
      t.string :name, null: false
      t.decimal :price, precision: 10, scale: 2
      t.boolean :active, default: true
      t.references :category, foreign_key: true
      t.timestamps
    end
  end
end
```


## Resources

- [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) — Complete guide to database migrations in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*