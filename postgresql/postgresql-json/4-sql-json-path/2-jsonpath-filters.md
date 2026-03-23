---
source_course: "postgresql-json"
source_lesson: "postgresql-json-jsonpath-filters"
---

# JSONPath Filter Expressions

## Introduction
Filters are the most powerful feature of JSONPath. They allow you to select JSON elements based on conditions — comparing values, matching strings, checking existence, and combining logic with AND/OR. Filters transform JSONPath from simple navigation into a full query language.

## Key Concepts
- **Filter syntax `? (condition)`**: Applies a boolean condition to filter elements. Only elements where the condition is true are returned.
- **`@` (current item)**: Inside a filter, `@` refers to the element currently being evaluated.
- **Comparison operators**: `==`, `!=`, `<`, `<=`, `>`, `>=` for value comparisons.
- **Logical operators**: `&&` (and), `||` (or), `!` (not) for combining conditions.
- **`like_regex`**: A string matching operator that supports regular expressions within filters.

## Real World Context
Filter expressions are essential for queries like "find all orders with a total above $100" or "find all users whose email matches a domain pattern." Without filters, you would need to expand arrays to rows, filter with WHERE, and re-aggregate — a verbose process. JSONPath filters do this in a single expression, often within a GIN-indexed `@?` check.

## Deep Dive

### Filter Syntax

Filters use the `? (condition)` syntax after a path element:

```sql
-- Filter syntax: ? (condition)
SELECT jsonb_path_query_array(
    '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]'::jsonb,
    '$[*] ? (@.age > 27)'
);
-- Result: [{"age": 30, "name": "Alice"}]
```

The `@` symbol refers to the current item being filtered. In `$[*] ? (@.age > 27)`, each array element is tested, and only those with age > 27 are returned.

### Common Filter Patterns

Here are the patterns you will use most often:

```sql
-- Equality filter
SELECT jsonb_path_query_array(
    '[{"status": "active"}, {"status": "inactive"}]'::jsonb,
    '$[*] ? (@.status == "active")'
);

-- Multiple conditions with && (and)
SELECT jsonb_path_query_array(
    '[{"name": "Alice", "age": 30, "active": true},
      {"name": "Bob", "age": 25, "active": true}]'::jsonb,
    '$[*] ? (@.age >= 25 && @.active == true)'
);

-- OR conditions with ||
SELECT jsonb_path_query_array(
    '[{"role": "admin"}, {"role": "user"}, {"role": "moderator"}]'::jsonb,
    '$[*] ? (@.role == "admin" || @.role == "moderator")'
);

-- Negation with !
SELECT jsonb_path_query_array(
    '[{"deleted": true}, {"deleted": false}]'::jsonb,
    '$[*] ? (!(@.deleted == true))'
);
```

These patterns combine to express complex filter logic concisely.

### String Operations in Filters

JSONPath supports string-specific operations:

```sql
-- starts with
SELECT jsonb_path_query_array(
    '[{"email": "alice@company.com"}, {"email": "bob@external.org"}]'::jsonb,
    '$[*] ? (@.email starts with "alice")'
);

-- like_regex for pattern matching
SELECT jsonb_path_query_array(
    '[{"email": "alice@company.com"}, {"email": "bob@company.com"}]'::jsonb,
    '$[*] ? (@.email like_regex "@company\\.com$")'
);

-- Case-insensitive regex
SELECT jsonb_path_query_array(
    '[{"name": "ALICE"}, {"name": "bob"}]'::jsonb,
    '$[*] ? (@.name like_regex "alice" flag "i")'
);
```

The `like_regex` operator uses POSIX regular expressions and supports flags like `"i"` for case-insensitive matching.

### exists() Filter Function

The `exists()` function checks whether a nested path exists:

```sql
-- Check if nested path exists
SELECT jsonb_path_query_array(
    '[{"user": {"email": "a@b.com"}}, {"user": {}}]'::jsonb,
    '$[*] ? (exists (@.user.email))'
);
-- Only returns objects where user.email exists
```

This is useful for filtering documents that have optional nested fields.

## Common Pitfalls
1. **Forgetting to double-escape backslashes in regex** — In SQL string literals, you need `\\\\` for a literal backslash in like_regex. Use dollar-quoted strings to reduce escaping.
2. **Using = instead of ==** — JSONPath uses `==` for equality comparison, not `=`. Using `=` causes a syntax error.
3. **Comparing different types without casting** — `@.age > "25"` compares a number to a string and may not behave as expected.

## Best Practices
1. **Combine filters with @? for indexed lookups** — `WHERE column @? '$.items[*] ? (@.price > 100)'` leverages GIN indexes.
2. **Use exists() for optional field checks** — It is more reliable than comparing to null.
3. **Keep filters simple** — Complex multi-condition filters can be hard to debug. Break them into separate checks when troubleshooting.

## Summary
- Filters use `? (condition)` syntax with `@` referring to the current element.
- Comparison operators (`==`, `!=`, `<`, `>`, etc.) and logical operators (`&&`, `||`, `!`) combine for complex conditions.
- `starts with` and `like_regex` provide string-specific filtering.
- `exists()` checks for the presence of nested paths.
- Filters work with `@?` and `@@` operators, which are GIN-indexable.

## Code Examples

**Filter expressions with comparisons, logical operators, and regex matching**

```sql
-- Filter array elements by condition
SELECT jsonb_path_query_array(
    '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]'::jsonb,
    '$[*] ? (@.age > 27)'
);
-- Result: [{"age": 30, "name": "Alice"}]

-- Multiple conditions with &&
SELECT jsonb_path_query_array(
    '[{"role": "admin", "active": true}, {"role": "user", "active": false}]'::jsonb,
    '$[*] ? (@.role == "admin" && @.active == true)'
);

-- Regex matching with like_regex
SELECT jsonb_path_query_array(
    '[{"email": "alice@company.com"}, {"email": "bob@external.org"}]'::jsonb,
    '$[*] ? (@.email like_regex "@company\\.com$")'
);
```


## Resources

- [JSONPath Filters Documentation](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*