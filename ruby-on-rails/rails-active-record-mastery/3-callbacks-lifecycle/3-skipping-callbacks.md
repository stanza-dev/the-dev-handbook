---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-skipping-callbacks"
---

# Suppressing and Skipping Callbacks

## Introduction

Sometimes you need to update the database without triggering callbacks — bulk imports, counter updates, or migrations that would fire thousands of side effects. Rails provides several methods to bypass callbacks, each with different trade-offs.

## Key Concepts

- **`update_column` / `update_columns`**: Update columns directly via SQL, skipping all callbacks and validations.
- **`insert_all` / `upsert_all`**: Bulk insert or upsert without instantiating models.
- **`delete` / `delete_all`**: Remove records via direct SQL, skipping callbacks.
- **`touch`**: Updates `updated_at` with a single SQL statement, skipping validations.

## Real World Context

A data migration updating 10 million rows should not trigger after_save callbacks that send emails. Counter caches need direct SQL to avoid infinite callback loops. Bulk CSV imports need `insert_all` for performance.

## Deep Dive

### update_column and update_columns

These issue a direct SQL UPDATE, bypassing the lifecycle:

```ruby
article = Article.find(1)
article.update_column(:views_count, article.views_count + 1)
article.update_columns(views_count: 100, last_viewed_at: Time.current)
```

No callbacks, validations, or `updated_at` touch occurs.

### insert_all and upsert_all

Bulk operations bypass models entirely:

```ruby
Article.insert_all([
  { title: "Post 1", body: "Content", created_at: Time.current, updated_at: Time.current },
  { title: "Post 2", body: "Content", created_at: Time.current, updated_at: Time.current }
])

Article.upsert_all(
  [{ id: 1, title: "Updated" }],
  unique_by: :id
)
```

These generate a single SQL statement, orders of magnitude faster than individual creates.

### Methods That Skip vs. Honor Callbacks

```ruby
# SKIP callbacks:
article.update_column(:title, "New")  # Direct SQL
Article.insert_all([...])              # Bulk SQL
Article.delete_all                     # Direct DELETE
article.delete                         # Direct DELETE

# HONOR callbacks:
article.update(title: "New")           # Full lifecycle
article.save                           # Full lifecycle
article.destroy                        # Runs destroy callbacks
Article.destroy_all                    # Instantiates each, runs callbacks
```

The key distinction: methods that bypass object instantiation skip callbacks.

## Common Pitfalls

- **Using `update_column` for regular updates**: It skips validations. Only use when you explicitly need to bypass the lifecycle.
- **Forgetting `updated_at` with `update_columns`**: These do not automatically touch `updated_at`. Include it manually if needed.
- **`delete_all` vs `destroy_all`**: `delete_all` skips callbacks. `destroy_all` honors them.

## Best Practices

- Use `insert_all` / `upsert_all` for bulk data imports.
- Use `update_column` only for performance-critical single-column updates.
- Prefer `destroy` over `delete` unless you specifically need to skip callbacks.

## Summary

- `update_column` / `update_columns` bypass all callbacks and validations.
- `insert_all` / `upsert_all` perform bulk operations without instantiating models.
- `delete` / `delete_all` skip callbacks; `destroy` / `destroy_all` honor them.
- Use callback-skipping methods for performance-critical operations.
- Always be intentional about which side effects you are bypassing.

## Code Examples

**Methods that skip callbacks (update_column, insert_all, delete) vs methods that honor them (update, save, destroy).**

```ruby
# Direct SQL update — skips callbacks and validations
article.update_column(:views_count, article.views_count + 1)

# Bulk insert — no models, no callbacks
Article.insert_all([
  { title: "Post 1", body: "Content", created_at: Time.current, updated_at: Time.current },
  { title: "Post 2", body: "Content", created_at: Time.current, updated_at: Time.current }
])

# delete skips callbacks, destroy honors them
article.delete    # Direct DELETE SQL
article.destroy   # Runs before/after_destroy callbacks
```


## Resources

- [Active Record Callbacks - Skipping](https://guides.rubyonrails.org/active_record_callbacks.html#skipping-callbacks) — Guide to methods that skip callbacks

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*