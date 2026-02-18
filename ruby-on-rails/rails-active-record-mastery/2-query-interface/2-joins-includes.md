---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-joins-includes"
---

# Joins, Includes, and Eager Loading

Understanding when to use `joins`, `includes`, `preload`, and `eager_load` is crucial for writing efficient queries.

## joins - For Filtering

Use `joins` when you need to filter by associated data but don't need to access it:

```ruby
# Find articles by published authors
Article.joins(:author).where(authors: { published: true })
# SELECT articles.* FROM articles
# INNER JOIN authors ON authors.id = articles.author_id
# WHERE authors.published = true

# Multiple joins
Article.joins(:author, :category)
Article.joins(comments: :author)  # Nested join

# Left outer join
Article.left_joins(:comments)
# SELECT articles.* FROM articles
# LEFT OUTER JOIN comments ON comments.article_id = articles.id
```

**Important**: `joins` doesn't load associated records into memory.

## includes - For Loading Associated Data

Use `includes` when you need to access associated data:

```ruby
# Eager load authors
articles = Article.includes(:author)
articles.each do |article|
  puts article.author.name  # No N+1 query!
end

# Rails decides the strategy:
# - Separate queries (like preload) when no filtering
# - JOIN (like eager_load) when filtering on association

# Multiple associations
Article.includes(:author, :comments)
Article.includes(comments: :author)  # Nested

# With conditions on association (forces JOIN)
Article.includes(:comments).where(comments: { approved: true })
```

## preload - Always Separate Queries

Forces separate queries for associations:

```ruby
Article.preload(:author, :comments)
# Query 1: SELECT * FROM articles
# Query 2: SELECT * FROM authors WHERE id IN (...)
# Query 3: SELECT * FROM comments WHERE article_id IN (...)
```

Use when:
- You don't filter by association
- You want predictable query behavior
- Separate queries are more efficient (large datasets)

## eager_load - Always JOIN

Forces a single query with LEFT OUTER JOIN:

```ruby
Article.eager_load(:author)
# SELECT articles.*, authors.*
# FROM articles
# LEFT OUTER JOIN authors ON authors.id = articles.author_id
```

Use when:
- You filter by association attributes
- You need all data in one query
- Dataset is small enough for a JOIN

## Comparison

| Method | Queries | Use When |
|--------|---------|----------|
| `joins` | 1 (INNER JOIN) | Filtering only |
| `includes` | 1 or 2+ | General eager loading |
| `preload` | 2+ (separate) | No filtering on association |
| `eager_load` | 1 (LEFT JOIN) | Filtering + loading |

## Practical Examples

```ruby
# Blog index page - load authors for display
@articles = Article.includes(:author).published.recent.limit(20)

# Filter by author AND display author
@articles = Article.eager_load(:author)
                   .where(authors: { verified: true })
                   .published

# Complex page with multiple associations
@article = Article.includes(
  :author,
  :tags,
  comments: [:author, :likes]
).find(params[:id])

# Count with joins (efficient)
Article.joins(:comments).group(:id).count
```

## Resources

- [Active Record Query Interface - Joins](https://guides.rubyonrails.org/active_record_querying.html#joining-tables) — Guide to joining tables in Active Record

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*