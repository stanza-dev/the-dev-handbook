---
source_course: "rails-foundations"
source_lesson: "rails-foundations-eager-loading"
---

# Eager Loading and N+1 Queries

## Introduction

The N+1 query problem is one of the most common performance issues in Rails applications. It happens silently, slowing down your pages without any error messages. Learning to detect and fix it is a critical Rails skill.

## Key Concepts

- **N+1 query problem**: When accessing an association in a loop triggers one query per record, resulting in N+1 total queries instead of 2.
- **Eager loading**: Loading associated records in advance with a single query, preventing N+1 issues.
- **`includes`**: The primary method for eager loading, which lets Rails choose the best strategy.
- **`preload`**: Forces separate queries with an IN clause. **`eager_load`**: Forces a single LEFT OUTER JOIN query.

## Real World Context

A page displaying 100 articles with their authors could execute 101 queries without eager loading, but only 2 with it. In production, N+1 queries are the number one cause of slow page loads. Tools like the Bullet gem help detect them automatically.

## Deep Dive

### The N+1 Problem

```ruby
# Controller
def index
  @articles = Article.all
end
```

```erb
<% @articles.each do |article| %>
  <p>By: <%= article.author.name %></p>  <!-- Triggers a query! -->
<% end %>
```

This generates:
```sql
SELECT * FROM articles;                    -- 1 query
SELECT * FROM authors WHERE id = 1;        -- N queries
SELECT * FROM authors WHERE id = 2;
-- ... one per article
```

### The Fix: includes

```ruby
def index
  @articles = Article.includes(:author)
end
```

Now only 2 queries:
```sql
SELECT * FROM articles;
SELECT * FROM authors WHERE id IN (1, 2, 3, ...);
```

### Eager Loading Methods

```ruby
# includes - Rails picks best strategy
Article.includes(:author)
Article.includes(:author, :tags)
Article.includes(comments: :author)  # Nested

# preload - Always separate queries
Article.preload(:comments)

# eager_load - Always LEFT OUTER JOIN
Article.eager_load(:comments)
```

Use `eager_load` when filtering by associated attributes:
```ruby
Article.eager_load(:comments).where(comments: { approved: true })
```

### Detecting N+1 Queries

```ruby
# Gemfile
gem 'bullet', group: :development

# config/environments/development.rb
config.after_initialize do
  Bullet.enable = true
  Bullet.alert = true
end
```

## Common Pitfalls

- **Not using includes on index pages**: Any page that loops over records and accesses associations needs eager loading.
- **Over-eager-loading**: Loading associations you don't use wastes memory. Only include what the view needs.
- **Using `eager_load` when `includes` suffices**: `eager_load` forces a JOIN which can be slower for large datasets.

## Best Practices

- Use `includes` for all index actions that display associated data.
- Install the Bullet gem in development to detect N+1 queries automatically.
- Watch your Rails log for repeated queries as a manual check.

## Summary

- The N+1 problem occurs when accessing associations in a loop triggers one query per record.
- `includes(:association)` eager loads data in 2 queries instead of N+1.
- Supports nested eager loading: `includes(comments: :author)`.
- Use the Bullet gem to detect N+1 queries in development.
- `preload` forces separate queries; `eager_load` forces a JOIN.

## Code Examples

**includes() eager loads associations in a single query, preventing the N+1 problem where accessing associations in a loop triggers one query per record.**

```ruby
# N+1 problem: 101 queries for 100 articles
@articles = Article.all
# Each article.author triggers a separate query

# Fix with includes: only 2 queries
@articles = Article.includes(:author)
# SELECT * FROM articles
# SELECT * FROM authors WHERE id IN (1, 2, 3, ...)
```


## Resources

- [Active Record Query Interface - Eager Loading](https://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations) — Guide to preventing N+1 queries

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*