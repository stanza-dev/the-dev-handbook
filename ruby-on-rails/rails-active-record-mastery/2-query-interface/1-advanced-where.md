---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-advanced-where"
---

# Advanced Where Clauses

## Introduction
The `where` method is the backbone of Active Record querying. Beyond simple equality checks, it supports ranges, pattern matching, NULL handling, OR/AND composition, and raw SQL fragments. Mastering these techniques lets you express almost any SQL condition without leaving Ruby.

## Key Concepts
- **`where.not`**: Negates conditions, producing `NOT IN` or `!=` SQL.
- **`.or`**: Combines two relations with SQL `OR` logic.
- **Range conditions**: Ruby ranges map to SQL `BETWEEN` or comparison operators.
- **`sanitize_sql_like`**: Escapes user input for LIKE queries to prevent injection of `%` and `_` wildcards.

## Real World Context
Every Rails application needs filtering: dashboards with date ranges, admin panels with multi-criteria search, APIs with complex query parameters. Knowing the full `where` API means you can express these filters cleanly without falling back to raw SQL strings.

## Deep Dive

### Comparison Operators with Ranges

```ruby
# Using ranges (more readable than raw SQL)
Product.where(price: 10..100)      # BETWEEN 10 AND 100
Product.where(price: 10...100)     # >= 10 AND < 100 (exclusive end)

# Using beginless/endless ranges (Ruby 2.7+)
Product.where(price: ..100)        # price <= 100
Product.where(price: 100..)        # price >= 100
```

Ranges are the idiomatic way to express comparison operators in Active Record. They generate efficient SQL and are easier to read than string conditions.

### NOT Conditions

```ruby
Article.where.not(published: false)
Article.where.not(author_id: nil)            # WHERE author_id IS NOT NULL
Article.where.not(status: ["draft", "archived"])  # NOT IN
```

### OR Conditions

```ruby
Article.where(published: true).or(Article.where(featured: true))
# WHERE published = true OR featured = true
```

Both sides of `.or` must be relations on the same model. You cannot mix different models.

### Pattern Matching (LIKE)

```ruby
# Using sanitize_sql_like for user input
query = Article.sanitize_sql_like(params[:search])
Article.where("title LIKE ?", "%#{query}%")

# Case-insensitive (PostgreSQL)
Article.where("title ILIKE ?", "%rails%")
```

Always use `sanitize_sql_like` when the search term comes from user input. Without it, a user could inject `%` or `_` wildcards.

### NULL Handling

```ruby
Article.where(published_at: nil)         # IS NULL
Article.where.not(published_at: nil)     # IS NOT NULL

# Rails 7+ missing/associated methods
Article.where.missing(:author)    # No associated author
Article.where.associated(:author) # Has associated author
```

### Date and Time Queries

```ruby
Article.where(created_at: Date.today.all_day)
Article.where(created_at: 1.week.ago..)
Article.where(published_at: start_date..end_date)
```

## Common Pitfalls
1. **SQL injection with string interpolation** -- Never write `where("name = '#{params[:name]}'")`. Always use placeholders: `where("name = ?", params[:name])`.
2. **Confusing `.or` scope** -- Both sides of `.or` must be on the same model. Trying to OR across different models raises an error.
3. **Hash conditions use AND** -- `where(a: 1, b: 2)` means `a = 1 AND b = 2`, not OR. Use `.or` for OR logic.

## Best Practices
1. **Prefer hash syntax** -- `where(status: "active")` is safer and more readable than `where("status = ?", "active")`.
2. **Use ranges for comparisons** -- `where(price: 10..100)` is cleaner than `where("price >= ? AND price <= ?", 10, 100)`.
3. **Sanitize LIKE inputs** -- Always use `sanitize_sql_like` for user-provided search terms.

## Summary
- `where` supports hashes, arrays, ranges, strings with placeholders, and raw SQL.
- Use `where.not` for negation, `.or` for OR conditions.
- Ranges map to `BETWEEN` and comparison operators.
- Always sanitize user input in LIKE queries.
- Rails 7+ adds `where.missing` and `where.associated` for association-based filtering.

## Code Examples

**Advanced where clause patterns -- ranges, OR, NOT, and NULL handling without raw SQL**

```ruby
# Range queries
Product.where(price: 10..100)   # BETWEEN 10 AND 100
Product.where(price: 100..)     # price >= 100

# OR conditions
Article.where(published: true).or(Article.where(featured: true))

# NOT conditions
User.where.not(role: ["guest", "banned"])

# NULL handling
Article.where.associated(:author)  # Has an author
```


## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Complete guide to querying with Active Record

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*