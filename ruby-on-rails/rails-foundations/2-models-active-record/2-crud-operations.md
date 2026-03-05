---
source_course: "rails-foundations"
source_lesson: "rails-foundations-crud-operations"
---

# CRUD Operations with Active Record

## Introduction

CRUD stands for Create, Read, Update, Delete — the four fundamental database operations. Active Record makes these operations feel natural in Ruby, turning SQL queries into intuitive method calls.

## Key Concepts

- **Create**: `new` + `save`, `create`, or `create!` to insert records into the database.
- **Read**: `find`, `find_by`, `where`, `all`, `first`, `last` to query records.
- **Update**: `update`, `update!`, or individual attribute assignment + `save`.
- **Delete**: `destroy` (runs callbacks) vs `delete` (skips callbacks).

## Real World Context

Every web application revolves around CRUD. A blog creates articles, reads them for display, updates them when edited, and deletes them when removed. Understanding Active Record's CRUD methods is essential for building any Rails application.

## Deep Dive

### Create

```ruby
# new + save (two steps)
article = Article.new(title: "My Title", body: "Content")
article.save  # => true or false

# create (one step, returns object)
article = Article.create(title: "My Title", body: "Content")

# create! (raises exception on failure)
article = Article.create!(title: "My Title", body: "Content")
```

### Read

```ruby
# Find by ID (raises RecordNotFound if missing)
article = Article.find(1)

# Find by attributes (returns nil if missing)
article = Article.find_by(title: "My Title")

# Collections
articles = Article.all
first = Article.first
last = Article.last

# Filtering with where
published = Article.where(published: true)
recent = Article.where("created_at > ?", 1.week.ago)
```

### Update

```ruby
article = Article.find(1)
article.update(title: "New Title")  # => true or false
article.update!(title: "New Title") # raises on failure

# Update many
Article.where(published: false).update_all(published: true)
```

### Delete

```ruby
article = Article.find(1)
article.destroy   # Runs callbacks
article.delete    # Skips callbacks (faster)

Article.where(published: false).destroy_all
```

### Persistence Checks

```ruby
article = Article.new(title: "Draft")
article.new_record?  # => true
article.persisted?   # => false

article.save
article.persisted?   # => true
```

## Common Pitfalls

- **Using `create` when you need error handling**: `create` returns the object even if it fails. Use `create!` or check `.valid?` / `.persisted?`.
- **Using `delete` instead of `destroy`**: `delete` skips callbacks and validations. Use `destroy` unless you need raw speed and understand the consequences.
- **Forgetting `where` is lazy**: `Article.where(...)` returns a relation, not results. It only hits the database when you iterate or call `.to_a`.

## Best Practices

- Use `create!` and `update!` in seeds and scripts where failures should be loud.
- Use `create` and `update` in controllers where you handle failures gracefully.
- Prefer `destroy` over `delete` to ensure callbacks (like cleaning up associations) run.

## Summary

- **Create**: `Article.create(attrs)` or `Article.new(attrs)` + `.save`.
- **Read**: `find`, `find_by`, `where`, `all`, `first`, `last`.
- **Update**: `.update(attrs)` or assign attributes and call `.save`.
- **Delete**: `.destroy` runs callbacks; `.delete` skips them.
- Use bang methods (`create!`, `update!`) when failures should raise exceptions.

## Code Examples

**The four CRUD operations in Active Record: create, find/where for reading, update for modifying, and destroy for removing records.**

```ruby
# Create
article = Article.create(title: "Hello", body: "World")

# Read
Article.find(1)                           # by ID
Article.find_by(title: "Hello")           # by attribute
Article.where(published: true)            # filtered

# Update
article.update(title: "Updated")

# Delete
article.destroy
```


## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Complete guide to querying with Active Record

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*