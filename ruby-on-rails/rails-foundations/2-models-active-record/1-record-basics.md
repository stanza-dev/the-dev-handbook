---
source_course: "rails-foundations"
source_lesson: "rails-foundations-active-record-basics"
---

# Introduction to Active Record

## Introduction

Active Record is Rails' Object-Relational Mapping (ORM) layer. It lets you interact with your database using Ruby objects instead of writing raw SQL, making database operations intuitive and productive.

## Key Concepts

- **ORM (Object-Relational Mapping)**: A technique that maps database tables to Ruby classes, rows to objects, and columns to attributes.
- **Active Record**: Rails' implementation of the ORM pattern, named after Martin Fowler's Active Record design pattern.
- **Migration**: A Ruby class that defines changes to your database schema in a version-controlled way.
- **Naming conventions**: Rails automatically maps class names to table names (e.g., `User` -> `users`, `LineItem` -> `line_items`).

## Real World Context

Without an ORM, you would write raw SQL for every database operation. Active Record lets you write `User.find(1)` instead of `SELECT * FROM users WHERE id = 1`. This makes code more readable, portable across databases, and less prone to SQL injection vulnerabilities.

## Deep Dive

### What an ORM Maps

```
Database Table: users          Ruby Class: User
+----+---------+-------------+  +---------------------+
| id |  name   |    email    |  | class User          |
+----+---------+-------------+  |   attr: id          |
| 1  | "Alice" | "a@test.com"|<-|   attr: name        |
| 2  | "Bob"   | "b@test.com"|  |   attr: email       |
+----+---------+-------------+  +---------------------+
```

### Creating Your First Model

```bash
bin/rails generate model Article title:string body:text
```

This creates a model file and a migration:

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  # Active Record handles everything
end
```

```ruby
# db/migrate/20240101000000_create_articles.rb
class CreateArticles < ActiveRecord::Migration[8.1]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :body
      t.timestamps
    end
  end
end
```

Run the migration:

```bash
bin/rails db:migrate
```

### Naming Conventions

| Model Class | Table Name | File Name |
|-------------|------------|------------|
| `Article` | `articles` | `article.rb` |
| `User` | `users` | `user.rb` |
| `LineItem` | `line_items` | `line_item.rb` |
| `Person` | `people` | `person.rb` |

### Using the Rails Console

```ruby
bin/rails console

article = Article.new(title: "Hello Rails", body: "My first article!")
article.save   # => true

article = Article.create(title: "Second Post", body: "More content")

Article.all    # All articles
Article.first  # First article
Article.find(1) # Find by ID
```

## Common Pitfalls

- **Forgetting to run migrations**: After generating a model, you must run `bin/rails db:migrate` before using it.
- **Fighting naming conventions**: Don't manually set table names unless absolutely necessary. Follow Rails conventions.
- **Not using the console**: The Rails console (`bin/rails c`) is the fastest way to test model behavior interactively.

## Best Practices

- Always use generators (`bin/rails generate model`) to create models and migrations together.
- Run `bin/rails db:migrate` immediately after creating a migration.
- Use `bin/rails console` to explore and test your models.

## Summary

- Active Record maps database tables to Ruby classes, rows to objects, and columns to attributes.
- Generate models with `bin/rails generate model Name field:type`.
- Migrations define schema changes; run them with `bin/rails db:migrate`.
- Rails naming conventions (e.g., `User` -> `users`) eliminate manual configuration.

## Code Examples

**A minimal Active Record model inherits from ApplicationRecord, gaining CRUD operations, querying, and database mapping automatically.**

```ruby
# Generate a model
# bin/rails generate model Article title:string body:text

class Article < ApplicationRecord
  # Inheriting from ApplicationRecord gives you
  # all Active Record functionality automatically
end

# In the Rails console:
article = Article.create(title: "Hello", body: "World")
Article.find(1)   # => #<Article id: 1, title: "Hello"...>
Article.all       # => all articles
```


## Resources

- [Active Record Basics](https://guides.rubyonrails.org/active_record_basics.html) — Official guide to Active Record fundamentals

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*