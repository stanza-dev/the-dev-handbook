---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-advanced-where"
---

# Advanced Where Clauses

The `where` method is incredibly powerful. Let's explore advanced querying techniques beyond simple equality checks.

## Comparison Operators

```ruby
# Greater than / Less than
Product.where("price > ?", 100)
Product.where("price < ?", 50)
Product.where("price >= ? AND price <= ?", 10, 100)

# Using ranges (more readable)
Product.where(price: 10..100)      # BETWEEN 10 AND 100
Product.where(price: 10...100)     # >= 10 AND < 100 (exclusive end)

# Using beginless/endless ranges (Ruby 2.7+)
Product.where(price: ..100)        # price <= 100
Product.where(price: 100..)        # price >= 100
```

## NOT Conditions

```ruby
# Using where.not
Article.where.not(published: false)
Article.where.not(author_id: nil)
Article.where.not(status: ["draft", "archived"])

# Combining with other conditions
Article.where(published: true).where.not(author_id: 1)
```

## OR Conditions

```ruby
# Using .or
Article.where(published: true).or(Article.where(featured: true))
# WHERE published = true OR featured = true

# Complex OR conditions
admin_articles = Article.where(author_id: admin_ids)
featured_articles = Article.where(featured: true)
Article.where(published: true).and(admin_articles.or(featured_articles))
```

## IN and NOT IN

```ruby
# Array automatically becomes IN
User.where(role: ["admin", "editor", "moderator"])
# WHERE role IN ('admin', 'editor', 'moderator')

User.where(id: [1, 2, 3, 4, 5])
# WHERE id IN (1, 2, 3, 4, 5)

# NOT IN
User.where.not(role: ["guest", "banned"])
# WHERE role NOT IN ('guest', 'banned')
```

## NULL Handling

```ruby
# Find NULL values
Article.where(published_at: nil)
# WHERE published_at IS NULL

# Find NOT NULL values
Article.where.not(published_at: nil)
# WHERE published_at IS NOT NULL

# Rails 7+ missing/present methods
Article.where.missing(:author)     # No associated author
Article.where.associated(:author)  # Has associated author
```

## Pattern Matching (LIKE)

```ruby
# Using LIKE with sanitization
Article.where("title LIKE ?", "%Rails%")
# WHERE title LIKE '%Rails%'

# Case-insensitive (PostgreSQL)
Article.where("title ILIKE ?", "%rails%")

# Using sanitize_sql_like for user input
query = Article.sanitize_sql_like(params[:search])
Article.where("title LIKE ?", "%#{query}%")
```

## Date and Time Queries

```ruby
# Today's articles
Article.where(created_at: Date.today.all_day)

# This week
Article.where(created_at: 1.week.ago..)

# Between dates
Article.where(published_at: start_date..end_date)

# Specific parts of dates
Article.where("EXTRACT(YEAR FROM created_at) = ?", 2024)
```

## Raw SQL (Use Carefully)

```ruby
# When Active Record isn't enough
Article.where("LENGTH(title) > ?", 50)
Article.where("created_at::date = ?", Date.today)  # PostgreSQL

# Named bind variables
Article.where("author_id = :author AND published = :pub",
              author: 1, pub: true)
```

## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Complete guide to querying with Active Record

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*