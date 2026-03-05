---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-schema-management"
---

# Schema Management and db:prepare

## Introduction

Rails provides several commands for managing your database schema. Understanding the difference between `db:migrate`, `db:setup`, `db:reset`, and `db:prepare` is essential for development, testing, and deployment.

## Key Concepts

- **`schema.rb`**: Auto-generated Ruby file representing the current database schema.
- **`structure.sql`**: Alternative that dumps raw SQL, preserving database-specific features.
- **`db:prepare`**: Creates the database if needed, loads schema, runs pending migrations, and seeds.
- **`db:schema:load`**: Loads the schema directly (faster than running all migrations).

## Real World Context

New team members run `db:prepare` to set up their database in one command. CI uses `db:schema:load` for speed. Production runs `db:migrate` for incremental changes.

## Deep Dive

### schema.rb vs structure.sql

By default, Rails generates `db/schema.rb`:

```ruby
ActiveRecord::Schema[8.1].define(version: 2025_03_01_120000) do
  create_table "users", force: :cascade do |t|
    t.string "email", null: false
    t.string "name"
    t.timestamps
    t.index ["email"], unique: true
  end
end
```

For database-specific features, switch to `structure.sql`:

```ruby
# config/application.rb
config.active_record.schema_format = :sql
```

### Database Commands Compared

```bash
bin/rails db:prepare       # All-in-one: create, load schema, migrate, seed
bin/rails db:setup          # Create, load schema, seed (no migrations)
bin/rails db:reset          # Drop, create, load schema, seed
bin/rails db:migrate        # Run pending migrations only
bin/rails db:schema:load    # Load schema.rb directly
bin/rails db:schema:dump    # Regenerate schema.rb from current database
```

Key differences:

| Command | Creates DB? | Loads Schema? | Migrates? | Seeds? |
|---------|:-----------:|:-------------:|:---------:|:------:|
| `db:prepare` | If needed | If needed | Yes | Yes |
| `db:setup` | Yes | Yes | No | Yes |
| `db:reset` | Yes (drops) | Yes | No | Yes |
| `db:migrate` | No | No | Yes | No |

### When to Use Each

```bash
# New developer setup
bin/rails db:prepare

# CI environment — fastest
bin/rails db:schema:load && bin/rails db:migrate

# Production deployment
bin/rails db:migrate

# Nuke and rebuild (development only!)
bin/rails db:reset
```

### Checking Migration Status

```bash
bin/rails db:migrate:status
#  Status   Migration ID    Migration Name
#    up     20250101120000  Create users
#   down    20250301120000  Add slug to articles   <- pending
```

## Common Pitfalls

- **Running `db:reset` in production**: This drops the database! Only use in development.
- **Editing `schema.rb` manually**: It is auto-generated. Your edits will be overwritten.
- **Not committing `schema.rb`**: Always commit it. It is the source of truth for `db:schema:load`.

## Best Practices

- Use `db:prepare` for onboarding new developers.
- Use `db:schema:load` + `db:migrate` in CI for speed.
- Always commit `schema.rb` to version control.

## Summary

- `db:prepare` is the all-in-one command for setting up a database.
- `db:schema:load` loads the schema file directly, faster than replaying migrations.
- `db:migrate` runs only pending migrations.
- `schema.rb` is auto-generated and should always be committed.
- Use `structure.sql` for database-specific features.

## Code Examples

**Common database commands — db:prepare for setup, db:schema:load for CI, db:migrate for production.**

```bash
# New developer setup
bin/rails db:prepare

# CI — fast schema load + pending migrations
bin/rails db:schema:load && bin/rails db:migrate

# Production — only new changes
bin/rails db:migrate

# Check status
bin/rails db:migrate:status
```


## Resources

- [Active Record Migrations - Schema Dumping](https://guides.rubyonrails.org/active_record_migrations.html#schema-dumping-and-you) — Official guide to schema management and db:prepare

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*