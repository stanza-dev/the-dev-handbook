---
source_course: "rails-security-hardening"
source_lesson: "rails-security-parameterized-queries"
---

# Parameterized Queries

## Introduction
Rails provides several ways to write safe, injection-proof queries. By using parameterized queries, you ensure that user input is always treated as data rather than executable SQL code.

## Key Concepts
- **Placeholder Syntax**: Using `?` in query strings to mark where safe parameter values should be inserted.
- **Hash Conditions**: Passing a hash to `where` so Rails handles all escaping automatically.
- **Named Placeholders**: Using symbols like `:name` in queries for clarity when the same value appears multiple times.

## Real World Context
Every production Rails application relies on parameterized queries for data safety. Whether you are building a search feature, a user lookup, or a complex report, hash conditions and placeholders are the tools you will use daily.

## Deep Dive

### Placeholder Syntax

Use `?` placeholders to safely insert values into query strings:

```ruby
# Safe — uses parameterized query
User.where("email = ?", params[:email])

# Multiple parameters
User.where(
  "created_at > ? AND role = ?",
  params[:since],
  params[:role]
)
```

Rails automatically escapes special characters in the parameter values, ensuring they are treated as data rather than SQL syntax.

### Hash Conditions (Preferred)

The safest and most readable approach is to pass a hash directly:

```ruby
# Hash conditions are always safe
User.where(email: params[:email])

# Multiple conditions
User.where(
  email: params[:email],
  active: true
)

# With ranges and arrays
Product.where(price: 10..50)
User.where(role: ['admin', 'moderator'])
```

Hash conditions are preferred because there is no way to accidentally introduce injection — Rails generates parameterized SQL behind the scenes.

### Named Placeholders

For complex queries where the same value appears multiple times, named placeholders improve readability:

```ruby
User.where(
  "first_name = :name OR last_name = :name",
  name: params[:search]
)
```

This avoids repeating the parameter and makes the intent clear.

### Safe Dynamic Columns

When column names come from user input, you need a whitelist approach since parameterization only protects values, not identifiers:

```ruby
# DANGEROUS — allows injection in column name
User.order("#{params[:sort]}")

# SAFE — whitelist allowed columns
ALLOWED_SORT_COLUMNS = %w[name email created_at]

def safe_sort(column)
  ALLOWED_SORT_COLUMNS.include?(column) ? column : 'created_at'
end

User.order(safe_sort(params[:sort]))
```

The whitelist ensures that only known, safe column names are used in the query.

## Common Pitfalls
1. **Forgetting to protect ORDER BY clauses** — Parameterized queries protect values in WHERE clauses, but column names in ORDER BY must be whitelisted separately.
2. **Using `find_by_sql` with interpolation** — Even `find_by_sql` needs array-style parameters: `User.find_by_sql(['SELECT * FROM users WHERE id = ?', params[:id]])`.

## Best Practices
1. **Default to hash conditions** — Use `where(column: value)` whenever possible. Fall back to placeholder syntax only for complex queries that hash conditions cannot express.
2. **Whitelist all dynamic identifiers** — Column names, table names, and sort directions must all be validated against a known-good list.

## Summary
- Use `?` placeholders or hash conditions for all user-supplied values in queries.
- Hash conditions are the safest and most readable option.
- Named placeholders with `:symbol` syntax help when values are reused in a query.
- Dynamic column names require a whitelist approach since parameterization only protects values.

## Code Examples

**Three safe query patterns in Rails — hash conditions, positional placeholders, and named placeholders all prevent SQL injection**

```ruby
# Hash conditions — safest and most idiomatic
User.where(email: params[:email], active: true)

# Placeholder syntax — for complex WHERE clauses
User.where("created_at > ? AND role = ?", params[:since], params[:role])

# Named placeholders — when a value appears multiple times
User.where("first_name = :q OR last_name = :q", q: params[:search])
```


## Resources

- [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) — Official Rails guide on building safe database queries

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*