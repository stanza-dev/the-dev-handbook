---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-understanding-n-plus-one"
---

# Understanding N+1 Queries

## Introduction
The N+1 query problem is the most common performance issue in Rails applications. It silently multiplies your database queries, turning a simple page load into hundreds of unnecessary round trips to the database.

## Key Concepts
- **N+1 Query**: A pattern where loading N records triggers N additional queries to fetch associated data, resulting in N+1 total queries instead of 2.
- **Eager Loading**: Pre-fetching associated records in a minimal number of queries to avoid N+1.
- **Lazy Loading**: Rails' default behavior of loading associations only when accessed, which causes N+1 in loops.

## Real World Context
A page displaying 50 blog posts with authors will fire 51 queries without eager loading — one for posts and one per author. In production with hundreds of records, this can add seconds to response times and overwhelm your database connection pool.

## Deep Dive
Consider this code that displays posts with their authors:

```ruby
# Controller
@posts = Post.limit(10)

# View
<% @posts.each do |post| %>
  <p><%= post.title %> by <%= post.author.name %></p>
<% end %>
```

This generates 11 queries for 10 posts:

```sql
SELECT * FROM posts LIMIT 10
SELECT * FROM authors WHERE id = 1
SELECT * FROM authors WHERE id = 2
-- ... 8 more queries
```

The fix is eager loading with `includes`:

```ruby
@posts = Post.includes(:author).limit(10)
```

Now only 2 queries run regardless of post count:

```sql
SELECT * FROM posts LIMIT 10
SELECT * FROM authors WHERE id IN (1, 2, 3, ...)
```

Use the `bullet` gem to detect N+1 queries in development:

```ruby
# Gemfile
gem 'bullet', group: :development

# config/environments/development.rb
config.after_initialize do
  Bullet.enable = true
  Bullet.alert = true
  Bullet.rails_logger = true
end
```

## Common Pitfalls
1. **Ignoring N+1 in views** — N+1 queries often hide in view templates where associations are accessed inside loops. Always check your logs for repeated queries.
2. **Over-eager loading** — Loading every association upfront wastes memory. Only eager load associations you actually use on that page.

## Best Practices
1. **Use bullet gem in development** — It alerts you to N+1 queries as you browse your app, catching issues before they reach production.
2. **Check your Rails logs** — Look for repeated SELECT statements with different IDs. This is the telltale sign of N+1.

## Summary
- N+1 queries fire one query per record in a loop, devastating performance.
- Use `includes` to eager load associations in just 2 queries.
- The `bullet` gem detects N+1 queries automatically in development.
- Always check Rails logs for repeated query patterns.

## Code Examples

**Before and after fixing an N+1 query — includes reduces 11 queries to just 2**

```ruby
# N+1 problem: 11 queries for 10 posts
@posts = Post.limit(10)
@posts.each { |post| post.author.name }

# Fixed: only 2 queries regardless of count
@posts = Post.includes(:author).limit(10)
@posts.each { |post| post.author.name }
```


## Resources

- [Eager Loading Associations](https://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations) — Official Rails guide on eager loading to prevent N+1 queries

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*