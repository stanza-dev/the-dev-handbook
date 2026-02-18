---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-understanding-n-plus-one"
---

# Understanding N Plus One

The N+1 query problem is one of the most common performance issues in Rails applications.

## What is N+1?

Consider this code that displays posts with their authors:

```ruby
# Controller
@posts = Post.limit(10)

# View
<% @posts.each do |post| %>
  <p><%= post.title %> by <%= post.author.name %></p>
<% end %>
```

This generates:
```sql
-- 1 query for posts
SELECT * FROM posts LIMIT 10

-- N queries for authors (one per post!)
SELECT * FROM authors WHERE id = 1
SELECT * FROM authors WHERE id = 2
SELECT * FROM authors WHERE id = 3
-- ... 7 more queries
```

That's 11 queries for 10 posts! With 1000 posts, you'd have 1001 queries.

## Detecting N+1 Queries

Add the `bullet` gem to catch N+1 queries in development:

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

## The Solution: Eager Loading

Use `includes` to load associations in one query:

```ruby
# Loads posts AND authors in 2 queries total
@posts = Post.includes(:author).limit(10)
```

Now generates:
```sql
SELECT * FROM posts LIMIT 10
SELECT * FROM authors WHERE id IN (1, 2, 3, ...)
```

Two queries regardless of how many posts you have!

See [Eager Loading Associations](https://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations).

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*