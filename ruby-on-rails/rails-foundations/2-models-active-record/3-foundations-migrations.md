---
source_course: "rails-foundations"
source_lesson: "rails-foundations-migrations"
---

# Database Migrations

Migrations are Ruby classes that let you modify your database schema over time. Instead of writing raw SQL, you use a Ruby DSL that's database-agnostic.

## Why Migrations?

- **Version control** for your database schema
- **Team collaboration** - share schema changes
- **Database agnostic** - same code works on SQLite, PostgreSQL, MySQL
- **Reversible** - roll back changes if needed

## Creating Migrations

### Generate a migration

```bash
# Create an empty migration
bin/rails generate migration AddPublishedToArticles

# Rails infers changes from the name
bin/rails generate migration AddPublishedToArticles published:boolean
```

### Migration file structure

```ruby
# db/migrate/20240101120000_add_published_to_articles.rb
class AddPublishedToArticles < ActiveRecord::Migration[8.0]
  def change
    add_column :articles, :published, :boolean, default: false
  end
end
```

## Common Migration Methods

### Creating tables

```ruby
def change
  create_table :products do |t|
    t.string :name, null: false
    t.text :description
    t.decimal :price, precision: 10, scale: 2
    t.integer :stock, default: 0
    t.boolean :active, default: true
    t.references :category, foreign_key: true

    t.timestamps  # created_at and updated_at
  end
end
```

### Adding columns

```ruby
def change
  add_column :articles, :published_at, :datetime
  add_column :articles, :author_name, :string, default: "Anonymous"
end
```

### Removing columns

```ruby
def change
  remove_column :articles, :author_name, :string
end
```

### Renaming columns

```ruby
def change
  rename_column :articles, :title, :headline
end
```

### Adding indexes

```ruby
def change
  add_index :articles, :published_at
  add_index :users, :email, unique: true
  add_index :orders, [:user_id, :created_at]  # Composite index
end
```

### Changing column types

```ruby
def change
  change_column :products, :price, :decimal, precision: 12, scale: 2
end
```

## Running Migrations

```bash
# Run all pending migrations
bin/rails db:migrate

# Rollback last migration
bin/rails db:rollback

# Rollback multiple migrations
bin/rails db:rollback STEP=3

# Check migration status
bin/rails db:migrate:status

# Reset database (drop, create, migrate)
bin/rails db:reset
```

## Column Types

Rails supports these column types:

| Type | Description |
|------|-------------|
| `:string` | Short text (VARCHAR) |
| `:text` | Long text (TEXT) |
| `:integer` | Whole numbers |
| `:bigint` | Large integers |
| `:float` | Floating point |
| `:decimal` | Precise decimals |
| `:boolean` | True/false |
| `:date` | Date only |
| `:datetime` | Date and time |
| `:time` | Time only |
| `:binary` | Binary data |
| `:json` | JSON data |

## The Schema File

After running migrations, Rails updates `db/schema.rb`:

```ruby
# db/schema.rb - Auto-generated, do not edit!
ActiveRecord::Schema[8.0].define(version: 2024_01_01_120000) do
  create_table "articles", force: :cascade do |t|
    t.string "title"
    t.text "body"
    t.boolean "published", default: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
end
```

This file represents the current state of your database.

## Resources

- [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) — Complete guide to database migrations in Rails

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*