---
source_course: "postgresql-json"
source_lesson: "jsonpath-filters"
---

# JSONPath Filter Expressions

Filters allow you to select JSON elements based on conditions.

## Filter Syntax

```sql
-- Filter syntax: ? (condition)
SELECT jsonb_path_query_array(
    '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]'::jsonb,
    '$[*] ? (@.age > 27)'
);
-- Result: [{"age": 30, "name": "Alice"}]

-- @ refers to the current item being filtered
-- Comparison operators: ==, !=, <, <=, >, >=
```

## Common Filter Patterns

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

## String Operations in Filters

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

## exists() Filter Function

```sql
-- Check if nested path exists
SELECT jsonb_path_query_array(
    '[{"user": {"email": "a@b.com"}}, {"user": {}}]'::jsonb,
    '$[*] ? (exists (@.user.email))'
);
-- Only returns objects where user.email exists
```

📖 [JSONPath Filter Expressions](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH)

## Resources

- [JSONPath Filters Documentation](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*