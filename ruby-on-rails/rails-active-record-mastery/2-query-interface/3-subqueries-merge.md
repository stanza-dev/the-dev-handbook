---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-subqueries-merge"
---

# Subqueries and Merging Scopes

## Introduction

Active Record can compose complex queries by nesting one query inside another (subqueries) and by combining scopes from different models using `merge`. These techniques let you build sophisticated queries while keeping each piece simple and testable.

## Key Concepts

- **Subquery**: A query nested inside another query, used in WHERE or FROM clauses.
- **`.merge()`**: Combines conditions from one scope/relation onto another, typically used with joins.
- **`.where(id: relation)`**: Active Record converts a relation to a subquery when used inside `where`.

## Real World Context

Real applications need queries like "find all users who have at least one published article." These require combining conditions across tables. Subqueries and merge keep these queries readable and composable.

## Deep Dive

### Subqueries with where

Active Record creates subqueries automatically when you pass a relation to `where`:

```ruby
published_author_ids = Article.where(published: true).select(:user_id)
User.where(id: published_author_ids)
# SQL: SELECT * FROM users WHERE id IN (SELECT user_id FROM articles WHERE published = true)
```

The inner query becomes a subquery in the SQL. This is more efficient than loading IDs into Ruby.

### NOT IN subqueries

```ruby
User.where.not(id: Article.select(:user_id))
# SQL: SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM articles)
```

### Merging Scopes Across Models

The `.merge()` method applies conditions from one model's scope when joining:

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :recent, -> { where("created_at > ?", 1.week.ago) }
end

User.joins(:articles).merge(Article.published).merge(Article.recent).distinct
# SQL: SELECT DISTINCT users.* FROM users
#      INNER JOIN articles ON articles.user_id = users.id
#      WHERE articles.published = true AND articles.created_at > '...'
```

### EXISTS Subqueries

For better performance with large datasets, use `EXISTS`:

```ruby
User.where(
  Article.where("articles.user_id = users.id").where(published: true).arel.exists
)
```

EXISTS stops searching as soon as it finds one match, which is faster than IN for large tables.

## Common Pitfalls

- **Loading IDs into memory**: `User.where(id: Article.pluck(:user_id))` loads all IDs into Ruby. Use `select` for a subquery instead.
- **Forgetting `.distinct` with joins**: Joining can produce duplicates. Use `.distinct` to deduplicate.
- **Merge conflicts**: If both sides set the same column condition, the merged scope wins.

## Best Practices

- Use subqueries (passing relations to `where`) instead of `pluck` + array.
- Use `merge` to compose scopes from different models with joins.
- Prefer `EXISTS` over `IN` for large-table membership tests.

## Summary

- Pass an Active Record relation to `where` to create an automatic subquery.
- Use `.merge()` to apply scopes from a joined model onto your query.
- `EXISTS` subqueries are faster than `IN` for large datasets.
- Avoid `pluck` + array — always prefer subqueries.
- Use `.distinct` after joins to avoid duplicate rows.

## Code Examples

**Subqueries keep filtering in the database, and merge composes scopes from different models in joined queries.**

```ruby
# Subquery — finds users with published articles
published_ids = Article.where(published: true).select(:user_id)
User.where(id: published_ids)
# SQL: WHERE id IN (SELECT user_id FROM articles WHERE published = true)

# Merge — applies Article scopes to a joined User query
User.joins(:articles).merge(Article.published).distinct
```


## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Official guide covering advanced querying including subqueries and scoping

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*