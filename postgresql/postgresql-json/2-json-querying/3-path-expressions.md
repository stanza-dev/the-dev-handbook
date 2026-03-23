---
source_course: "postgresql-json"
source_lesson: "postgresql-json-path-expressions"
---

# JSON Path Expressions

## Introduction
JSON Path is a powerful query language for JSONB data, standardized in SQL:2016 and supported in PostgreSQL 12+. It allows you to write expressive queries that filter, navigate, and extract data from complex JSON structures using a concise path syntax. JSON Path often replaces multiple chained extraction operators with a single expression.

## Key Concepts
- **`@?` operator**: Returns true if the JSON path expression matches any items in the JSONB document.
- **`@@` operator**: Returns true if the JSON path predicate evaluates to true.
- **jsonb_path_query()**: A function that returns all matching items as separate rows.
- **jsonb_path_query_array()**: A function that returns all matching items collected into a single JSONB array.
- **jsonb_path_exists()**: A function that returns a boolean indicating whether the path matches any items.

## Real World Context
JSON Path is especially useful when you need to filter elements within arrays nested inside documents. For example, an order management system might use JSON Path to find all orders containing a line item above a certain price, without expanding the array to rows. Analytics platforms use JSON Path to extract and filter nested event properties in a single expression.

## Deep Dive

### JSON Path Operators

PostgreSQL provides two operators for JSON Path:

```sql
-- @? : Returns true if path matches anything
SELECT '{"a": [1,2,3,4,5]}'::jsonb @? '$.a[*] ? (@ > 2)';
-- true (some elements > 2)

-- @@ : Returns predicate result
SELECT '{"a": [1,2,3,4,5]}'::jsonb @@ '$.a[*] > 2';
-- true
```

Both operators can use GIN indexes, making them efficient for large-scale filtering.

### JSON Path Syntax

The path language uses `$` for the root and dot notation for navigation:

```sql
-- Basic path navigation
$.store.book         -- Navigate to store.book
$.store.book[0]      -- First element
$.store.book[*]      -- All elements
$.store.book[-1:]    -- Last element

-- Filter expressions
$.store.book[*] ? (@.price < 10)    -- Books under $10
$.store.book[*] ? (@.author == "Herman")  -- By author
```

The `@` symbol refers to the current element being evaluated in the filter.

### JSON Path Functions

Use these functions when you need the actual matching values rather than a boolean:

```sql
-- jsonb_path_query: Returns all matching items as separate rows
SELECT jsonb_path_query(
    '{"a": [1,2,3,4,5]}'::jsonb,
    '$.a[*] ? (@ > 2)'
);
-- Returns: 3, 4, 5 (as separate rows)

-- jsonb_path_query_array: Returns as a single array
SELECT jsonb_path_query_array(
    '{"a": [1,2,3,4,5]}'::jsonb,
    '$.a[*] ? (@ > 2)'
);
-- Returns: [3, 4, 5]

-- jsonb_path_exists: Boolean check
SELECT jsonb_path_exists(
    '{"a": [1,2,3]}'::jsonb,
    '$.a[*] ? (@ > 5)'
);
-- Returns: false
```

Choose `jsonb_path_query` when you want to process each match as a row, and `jsonb_path_query_array` when you want all matches as one value.

### Variables in JSON Path

You can pass dynamic values into JSON Path expressions using the third argument:

```sql
SELECT jsonb_path_query(
    '{"a": [1,2,3,4,5]}'::jsonb,
    '$.a[*] ? (@ >= $min && @ <= $max)',
    '{"min": 2, "max": 4}'
);
-- Returns: 2, 3, 4
```

Variables make JSON Path expressions reusable and safe from injection.

## Common Pitfalls
1. **Confusing @? with @@** — `@?` checks if the path returns any items, while `@@` evaluates a predicate and returns its boolean result. They are not interchangeable.
2. **Expecting strict mode by default** — JSON Path uses lax mode by default, which silently returns no results for missing keys instead of raising errors.
3. **Forgetting to quote string comparisons** — In JSON Path filters, strings must be double-quoted: `@.name == "Alice"`, not `@.name == Alice`.

## Best Practices
1. **Use @? in WHERE clauses with GIN indexes** — The `@?` operator is GIN-indexable and gives you powerful filtering with concise syntax.
2. **Prefer variables over string concatenation** — Always pass dynamic values through the variables parameter to avoid injection and improve readability.
3. **Use jsonb_path_query_array for subquery results** — When you want filtered nested data as a column, `jsonb_path_query_array` keeps results as a single value per row.

## Summary
- JSON Path provides a concise query language for navigating and filtering JSONB data.
- `@?` and `@@` operators can be used in WHERE clauses and are GIN-indexable.
- `jsonb_path_query`, `jsonb_path_query_array`, and `jsonb_path_exists` return matching data in different formats.
- Variables (the third argument) allow dynamic, injection-safe filtering.
- JSON Path uses lax mode by default, silently handling missing keys.

## Code Examples

**Advanced JSON Path Queries**

```sql
-- Find all products where any spec value exceeds threshold
SELECT * FROM products
WHERE attributes @? '$.specs.* ? (@ > 500)';

-- Extract matching items
SELECT 
    id,
    jsonb_path_query_array(
        data,
        '$.items[*] ? (@.status == "pending")'
    ) AS pending_items
FROM orders;

-- Use variables for dynamic filtering
SELECT * FROM events
WHERE jsonb_path_exists(
    data,
    '$ ? (@.amount > $threshold)',
    jsonb_build_object('threshold', 100)
);
```


## Resources

- [SQL/JSON Path Language](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*