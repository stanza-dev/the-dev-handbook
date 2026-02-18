---
source_course: "rails-foundations"
source_lesson: "rails-foundations-active-record-basics"
---

# Introduction to Active Record

Active Record is Rails' **Object-Relational Mapping (ORM)** layer. It connects your Ruby objects to database tables, allowing you to work with data using Ruby instead of SQL.

## What is an ORM?

An ORM maps:
- **Database tables** → Ruby classes (Models)
- **Table rows** → Ruby objects (Instances)
- **Table columns** → Object attributes

```
Database Table: users          Ruby Class: User
┌────┬─────────┬─────────────┐  ┌─────────────────────┐
│ id │  name   │    email    │  │ class User          │
├────┼─────────┼─────────────┤  │   attr: id          │
│ 1  │ "Alice" │ "a@test.com"│◄─┤   attr: name        │
│ 2  │ "Bob"   │ "b@test.com"│  │   attr: email       │
└────┴─────────┴─────────────┘  └─────────────────────┘
```

## Creating Your First Model

Generate a model using the Rails generator:

```bash
bin/rails generate model Article title:string body:text
```

This creates:
1. **Model file**: `app/models/article.rb`
2. **Migration file**: `db/migrate/XXXXXX_create_articles.rb`

### The Model Class

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  # That's it! Active Record handles the rest
end
```

By inheriting from `ApplicationRecord`, your class gains all Active Record functionality.

### The Migration

```ruby
# db/migrate/20240101000000_create_articles.rb
class CreateArticles < ActiveRecord::Migration[8.0]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :body

      t.timestamps  # Adds created_at and updated_at
    end
  end
end
```

Run the migration to create the table:

```bash
bin/rails db:migrate
```

## Naming Conventions

Active Record uses **conventions** to map classes to tables:

| Model Class | Table Name | File Name |
|-------------|------------|------------|
| `Article` | `articles` | `article.rb` |
| `User` | `users` | `user.rb` |
| `LineItem` | `line_items` | `line_item.rb` |
| `Person` | `people` | `person.rb` |

Rails automatically:
- Pluralizes class names for table names
- Uses snake_case for multi-word tables
- Handles irregular plurals (person → people)

## The Rails Console

Test your models interactively:

```bash
bin/rails console
```

In the console:

```ruby
# Create a new article
article = Article.new(title: "Hello Rails", body: "My first article!")
article.save
# => true

# Or create in one step
article = Article.create(title: "Second Post", body: "More content")

# Find articles
Article.all          # All articles
Article.first        # First article
Article.find(1)      # Find by ID
```

## Resources

- [Active Record Basics](https://guides.rubyonrails.org/active_record_basics.html) — Official guide to Active Record fundamentals

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*