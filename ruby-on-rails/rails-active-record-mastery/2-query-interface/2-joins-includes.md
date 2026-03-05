---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-joins-includes"
---

# Joins, Includes, and Eager Loading

## Introduction
Understanding when to use `joins`, `includes`, `preload`, and `eager_load` is crucial for writing efficient queries. Choosing the wrong method leads to N+1 queries or unnecessarily heavy JOINs. This lesson breaks down each method so you can pick the right tool every time.

## Key Concepts
- **`joins`**: Produces an INNER JOIN for filtering; does NOT load associated records into memory.
- **`includes`**: Eager-loads associations to prevent N+1 queries; Rails chooses the strategy (separate queries or JOIN).
- **`preload`**: Always uses separate queries for eager loading.
- **`eager_load`**: Always uses a single LEFT OUTER JOIN for eager loading.
- **N+1 problem**: Executing one query to load N parent records, then N additional queries to load each parent's associations.

## Real World Context
N+1 queries are the most common performance problem in Rails applications. A blog index page listing 20 articles with author names can fire 21 queries instead of 2. In production, this compounds quickly: 20 articles with authors, categories, and comment counts could mean 80+ queries per page load.

## Deep Dive

### joins -- For Filtering

Use `joins` when you need to filter by associated data but do not need to access it:

```ruby
Article.joins(:author).where(authors: { published: true })
# SELECT articles.* FROM articles
# INNER JOIN authors ON authors.id = articles.author_id
# WHERE authors.published = true

# Multiple joins
Article.joins(:author, :category)
Article.joins(comments: :author)  # Nested join

# Left outer join
Article.left_joins(:comments)
```

Key insight: `joins` does not load associated records. If you access `article.author` after a `joins` query, Rails fires a separate query.

### includes -- For Loading Associated Data

```ruby
articles = Article.includes(:author)
articles.each do |article|
  puts article.author.name  # No N+1 query!
end

# Multiple associations
Article.includes(:author, :comments)
Article.includes(comments: :author)  # Nested
```

Rails decides the strategy: separate queries when there is no filtering on the association, or a JOIN when you filter on the associated table.

### preload vs eager_load

```ruby
# preload: Always separate queries
Article.preload(:author, :comments)
# Query 1: SELECT * FROM articles
# Query 2: SELECT * FROM authors WHERE id IN (...)
# Query 3: SELECT * FROM comments WHERE article_id IN (...)

# eager_load: Always LEFT OUTER JOIN
Article.eager_load(:author)
# SELECT articles.*, authors.*
# FROM articles
# LEFT OUTER JOIN authors ON authors.id = articles.author_id
```

### Comparison Table

| Method | Queries | Use When |
|--------|---------|----------|
| `joins` | 1 (INNER JOIN) | Filtering only |
| `includes` | 1 or 2+ | General eager loading |
| `preload` | 2+ (separate) | No filtering on association |
| `eager_load` | 1 (LEFT JOIN) | Filtering + loading |

## Common Pitfalls
1. **Using `includes` when you only need to filter** -- `includes` loads all associated data into memory. If you only need to filter by an association, use `joins` instead.
2. **Forgetting to eager-load in loops** -- Any time you iterate over records and access an association, you risk N+1 queries. Use `includes` proactively.
3. **Using `preload` with association conditions** -- `preload` cannot filter on associations because it uses separate queries. Use `eager_load` or `includes` when filtering.

## Best Practices
1. **Use `includes` by default** -- It handles most cases correctly and lets Rails optimize the strategy.
2. **Use `joins` for pure filtering** -- When you do not need the associated data, `joins` is lighter.
3. **Use Bullet gem in development** -- The `bullet` gem automatically detects N+1 queries and suggests `includes` calls.

## Summary
- `joins` filters by associations without loading them (INNER JOIN).
- `includes` eager-loads associations to prevent N+1 queries.
- `preload` forces separate queries; `eager_load` forces a LEFT JOIN.
- Use `joins` for filtering, `includes` for display, and `eager_load` when you need both.
- Install the Bullet gem to catch N+1 queries during development.

## Code Examples

**Solving N+1 queries with includes and using joins for efficient filtering**

```ruby
# N+1 problem
articles = Article.all
articles.each { |a| puts a.author.name }  # 1 + N queries!

# Fixed with includes
articles = Article.includes(:author)
articles.each { |a| puts a.author.name }  # 2 queries total

# Filtering with joins (no data loading)
Article.joins(:author).where(authors: { verified: true })
```


## Resources

- [Active Record Query Interface - Joins](https://guides.rubyonrails.org/active_record_querying.html#joining-tables) — Guide to joining tables in Active Record

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*