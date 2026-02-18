---
source_course: "postgresql-json"
source_lesson: "json-path-expressions"
---

# JSON Path Expressions

JSON Path is a powerful query language for JSONB data, introduced in PostgreSQL 12 and enhanced in later versions.

## JSON Path Operators

| Operator | Description |
|----------|-------------|
| `@?` | Does path return any items? |
| `@@` | Path predicate returns true? |

```sql
-- @? : Returns true if path matches anything
SELECT '{"a": [1,2,3,4,5]}'::jsonb @? '$.a[*] ? (@ > 2)';
-- true (some elements > 2)

-- @@ : Returns predicate result
SELECT '{"a": [1,2,3,4,5]}'::jsonb @@ '$.a[*] > 2';
-- true
```

## JSON Path Syntax

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

## JSON Path Functions

```sql
-- jsonb_path_query: Returns all matching items
SELECT jsonb_path_query(
    '{"a": [1,2,3,4,5]}'::jsonb,
    '$.a[*] ? (@ > 2)'
);
-- Returns: 3, 4, 5 (as separate rows)

-- jsonb_path_query_array: Returns as array
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

## Variables in JSON Path

```sql
SELECT jsonb_path_query(
    '{"a": [1,2,3,4,5]}'::jsonb,
    '$.a[*] ? (@ >= $min && @ <= $max)',
    '{"min": 2, "max": 4}'
);
-- Returns: 2, 3, 4
```

## Code Examples

```undefined
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
WHERE data @@ jsonb_path_query_first(
    '$.timestamp > $cutoff',
    jsonb_build_object('cutoff', '2024-01-01')
)::boolean;
```


## Resources

- [SQL/JSON Path Language](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*