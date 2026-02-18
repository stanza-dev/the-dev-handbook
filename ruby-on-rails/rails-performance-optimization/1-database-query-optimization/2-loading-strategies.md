---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-eager-loading-strategies"
---

# Eager Loading Strategies

Rails provides three methods for eager loading, each with different behaviors.

## includes (Smart Choice)

Rails chooses the best strategy based on query conditions:

```ruby
# Two separate queries (default)
Post.includes(:author)
# SELECT * FROM posts
# SELECT * FROM authors WHERE id IN (...)

# Single JOIN query (when filtering on association)
Post.includes(:author).where(authors: { active: true })
# SELECT * FROM posts
# LEFT OUTER JOIN authors ON ...
# WHERE authors.active = true
```

## preload (Always Separate Queries)

Forces separate queries, useful when you don't need to filter:

```ruby
Post.preload(:author, :comments)
# Query 1: SELECT * FROM posts
# Query 2: SELECT * FROM authors WHERE id IN (...)
# Query 3: SELECT * FROM comments WHERE post_id IN (...)
```

## eager_load (Always JOIN)

Forces a single JOIN query:

```ruby
Post.eager_load(:author)
# SELECT posts.*, authors.* 
# FROM posts 
# LEFT OUTER JOIN authors ON authors.id = posts.author_id
```

## Nested Eager Loading

Load multiple levels of associations:

```ruby
# Load posts -> comments -> author
Post.includes(comments: :author)

# Multiple associations at same level
Post.includes(:author, :category, :tags)

# Complex nesting
Post.includes(
  :author,
  comments: [:author, :likes],
  tags: :category
)
```

## When to Use Each

| Method | Use When |
|--------|----------|
| `includes` | Default choice, let Rails decide |
| `preload` | Loading lots of data, avoiding large JOINs |
| `eager_load` | Need to filter/order by association columns |

```ruby
# Filtering by association - use includes or eager_load
Post.includes(:author).where(authors: { verified: true })

# Just displaying data - preload is fine
Post.preload(:author, :comments).limit(20)
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*