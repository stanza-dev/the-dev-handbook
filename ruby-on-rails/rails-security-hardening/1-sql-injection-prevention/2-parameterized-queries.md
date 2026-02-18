---
source_course: "rails-security-hardening"
source_lesson: "rails-security-parameterized-queries"
---

# Parameterized Queries

Rails provides several ways to write safe, injection-proof queries.

## Placeholder Syntax

Use `?` placeholders to safely insert values:

```ruby
# Safe - uses parameterized query
User.where("email = ?", params[:email])

# Multiple parameters
User.where(
  "created_at > ? AND role = ?",
  params[:since],
  params[:role]
)
```

Rails automatically escapes special characters, treating input as data, not SQL.

## Hash Conditions (Preferred)

The safest and most readable approach:

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

## Named Placeholders

For complex queries with repeated values:

```ruby
User.where(
  "first_name = :name OR last_name = :name",
  name: params[:search]
)
```

## Safe Dynamic Columns

When column names come from user input:

```ruby
# DANGEROUS - allows injection in column name
User.order("#{params[:sort]}")

# SAFE - whitelist allowed columns
ALLOWED_SORT_COLUMNS = %w[name email created_at]

def safe_sort(column)
  ALLOWED_SORT_COLUMNS.include?(column) ? column : 'created_at'
end

User.order(safe_sort(params[:sort]))
```

## find_by Safety

```ruby
# Safe
User.find_by(email: params[:email])

# Safe with multiple conditions
User.find_by(email: params[:email], active: true)
```

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*