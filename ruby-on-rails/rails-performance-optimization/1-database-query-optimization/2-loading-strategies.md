---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-eager-loading-strategies"
---

# Eager Loading Strategies

## Introduction
Rails provides three methods for eager loading associations: `includes`, `preload`, and `eager_load`. Each uses a different SQL strategy, and choosing the right one depends on whether you need to filter or just display associated data.

## Key Concepts
- **includes**: Smart method that lets Rails choose between separate queries or a JOIN based on your query conditions.
- **preload**: Always uses separate queries — one per association.
- **eager_load**: Always uses a single LEFT OUTER JOIN query.

## Real World Context
In a production dashboard showing orders with customers and line items, choosing `preload` over `eager_load` can cut memory usage in half by avoiding massive JOIN result sets. The right strategy depends on your query pattern.

## Deep Dive

### includes (Default Choice)

Rails automatically picks the best strategy:

```ruby
# Two separate queries (default behavior)
Post.includes(:author)
# SELECT * FROM posts
# SELECT * FROM authors WHERE id IN (...)

# Single JOIN when filtering on the association
Post.includes(:author).where(authors: { active: true })
# SELECT * FROM posts LEFT OUTER JOIN authors ON ...
# WHERE authors.active = true
```

### preload (Always Separate Queries)

```ruby
Post.preload(:author, :comments)
# Query 1: SELECT * FROM posts
# Query 2: SELECT * FROM authors WHERE id IN (...)
# Query 3: SELECT * FROM comments WHERE post_id IN (...)
```

### eager_load (Always JOIN)

```ruby
Post.eager_load(:author)
# SELECT posts.*, authors.*
# FROM posts
# LEFT OUTER JOIN authors ON authors.id = posts.author_id
```

### Nested Eager Loading

```ruby
Post.includes(comments: :author)
Post.includes(:author, :category, :tags)
Post.includes(:author, comments: [:author, :likes], tags: :category)
```

| Method | Use When |
|--------|----------|
| `includes` | Default choice — let Rails decide |
| `preload` | Loading lots of data, avoiding large JOINs |
| `eager_load` | Need to filter/order by association columns |

## Common Pitfalls
1. **Using eager_load for display-only pages** — JOINs create large result sets with duplicated data. Use `preload` when you just need to display associated records.
2. **Forgetting nested associations** — Loading `Post.includes(:comments)` but then accessing `comment.author` in the view still causes N+1. Include the full chain: `includes(comments: :author)`.

## Best Practices
1. **Start with includes** — It handles most cases correctly. Only switch to `preload` or `eager_load` when you have a specific reason.
2. **Use strict_loading in development** — Enable `strict_loading` to raise errors on lazy-loaded associations: `Post.strict_loading.includes(:author)`.

## Summary
- `includes` is the default — Rails picks the best SQL strategy automatically.
- `preload` forces separate queries, ideal for large data sets.
- `eager_load` forces a JOIN, required when filtering by association columns.
- Always include nested associations you access in views.

## Code Examples

**Three eager loading strategies — includes adapts automatically, preload avoids JOINs, eager_load forces a JOIN**

```ruby
# Let Rails choose the strategy
Post.includes(:author).where(authors: { verified: true })

# Force separate queries for large datasets
Post.preload(:author, :comments).limit(100)

# Force JOIN when you must filter by association
Post.eager_load(:author).order('authors.name ASC')
```


## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Official Rails guide covering includes, preload, and eager_load

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*