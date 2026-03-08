---
source_course: "rails-security-hardening"
source_lesson: "rails-security-safe-query-syntax"
---

# Advanced Query Safety

## Introduction
Beyond basic parameterization, Rails developers encounter edge cases that require extra care: LIKE queries with wildcards, raw SQL fragments, and dynamic table references. This lesson covers these advanced scenarios.

## Key Concepts
- **LIKE Wildcards**: Special characters `%` and `_` in LIKE queries that need separate handling from SQL injection protection.
- **Arel**: Rails' internal SQL AST builder that provides a fully safe, programmatic way to construct complex queries.
- **sanitize_sql_like**: A Rails method that escapes LIKE-specific wildcard characters.

## Real World Context
Search features, autocomplete, and filtering interfaces frequently use LIKE queries. Getting these right means your search works correctly without opening injection vulnerabilities.

## Deep Dive

### Safe LIKE Queries

When building search features, you need to handle both SQL injection and LIKE wildcards:

```ruby
# Safe from injection but wildcards in user input could match unexpectedly
Product.where("name LIKE ?", "%#{params[:q]}%")

# Fully safe — also escapes LIKE wildcards in user input
Product.where("name LIKE ?", "%#{sanitize_sql_like(params[:q])}%")
```

The `sanitize_sql_like` method escapes `%`, `_`, and `\` characters in user input so they are treated literally rather than as wildcards.

### Using Arel for Complex Queries

Arel provides a programmatic, injection-safe way to build queries:

```ruby
users = User.arel_table
query = users[:email].matches("%#{sanitize_sql_like(params[:q])}%")
User.where(query)
```

Arel generates parameterized SQL automatically, making it impossible to introduce injection through its API.

### Safe Raw SQL with Arrays

When you must use raw SQL, always use array-style parameterization:

```ruby
# Safe — parameterized raw SQL
results = ActiveRecord::Base.connection.exec_query(
  "SELECT * FROM users WHERE email = $1 AND active = $2",
  "SQL",
  [[nil, params[:email]], [nil, true]]
)
```

This approach passes values separately from the query string, preventing injection.

## Common Pitfalls
1. **Forgetting to escape LIKE wildcards** — A user searching for `100%` could match far more rows than expected if `%` is not escaped.
2. **Using `connection.execute` with interpolation** — Low-level database methods have no automatic protection. Always parameterize.

## Best Practices
1. **Use `sanitize_sql_like` for all LIKE queries** — It costs nothing and prevents wildcard injection.
2. **Prefer Arel over raw SQL** — Arel is harder to read at first but eliminates entire categories of injection bugs.

## Summary
- LIKE queries need `sanitize_sql_like` in addition to parameterization.
- Arel provides a fully safe, programmatic way to build complex queries.
- Raw SQL must always use array-style or positional parameter binding.
- Never use string interpolation with any database method.

## Code Examples

**Advanced safe query techniques — sanitize_sql_like for LIKE wildcards and Arel for programmatic query building**

```ruby
# Escaping LIKE wildcards in search queries
search_term = sanitize_sql_like(params[:q])
Product.where("name LIKE ?", "%#{search_term}%")

# Using Arel for type-safe query building
users = User.arel_table
User.where(users[:role].eq('admin').and(users[:active].eq(true)))
```


## Resources

- [Active Record Query Interface — Conditions](https://guides.rubyonrails.org/active_record_querying.html#conditions) — Official Rails guide section on building query conditions safely

---

> 📘 *This lesson is part of the [Rails Security Hardening](https://stanza.dev/courses/rails-security-hardening) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*