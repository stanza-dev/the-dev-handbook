---
source_course: "postgresql-json"
source_lesson: "postgresql-json-hybrid-design"
---

# Hybrid Relational and Document Design

## Introduction
PostgreSQL excels at combining relational and document models in the same table. The key is knowing when to use each approach: relational columns for structured, frequently queried data with constraints, and JSONB for flexible, variable-schema attributes. This hybrid pattern gives you the best of both worlds.

## Key Concepts
- **Hybrid schema**: A table design that uses relational columns for core, structured fields and JSONB columns for flexible, variable attributes.
- **Variable schema data**: Data where the set of fields varies between records, such as product attributes that differ by category.
- **Sparse columns problem**: When most rows have NULL for most optional columns, JSONB is more efficient than adding many nullable relational columns.
- **CHECK constraints on JSONB**: Database-level enforcement that the JSONB column contains the expected top-level type and required keys.

## Real World Context
E-commerce platforms are the classic example: every product has a name, price, and category (relational), but a laptop has RAM and CPU specs while a shirt has size and color (JSONB attributes). Healthcare systems store standard patient demographics as columns but use JSONB for variable clinical observations. This pattern avoids the proliferation of nullable columns while maintaining strong constraints on core data.

## Deep Dive

### When to Use JSONB

Use JSONB for data that is:

1. **Variable schemas**: Product attributes, form responses
2. **Nested data**: Address components, metadata
3. **External API data**: Preserve original structure
4. **Sparse columns**: Only store what is needed
5. **Rapidly evolving schemas**: Avoid migrations

### When to Use Relational Columns

Use relational columns for data that needs:

1. **Frequent joins**: Data referenced by other tables
2. **Unique constraints**: Email, username
3. **Foreign keys**: Referential integrity
4. **Aggregations**: SUM, AVG on columns
5. **Strict validation**: Data must conform to a fixed schema

### Hybrid Pattern Example

Here is a well-designed hybrid schema:

```sql
CREATE TABLE products (
    -- Relational: Core, frequently queried fields
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    category_id INTEGER REFERENCES categories(id),
    created_at TIMESTAMPTZ DEFAULT now(),
    
    -- Document: Variable attributes per product type
    attributes JSONB DEFAULT '{}',
    
    -- Constraints on JSON
    CONSTRAINT valid_attributes CHECK (jsonb_typeof(attributes) = 'object')
);

-- Indexes for both paradigms
CREATE INDEX idx_products_category ON products (category_id);
CREATE INDEX idx_products_attrs ON products USING GIN (attributes);
```

The relational columns handle core business logic, while the JSONB column handles variable attributes.

### Query Patterns

Combine relational and JSON filtering in a single query:

```sql
-- Combine relational and JSON filtering
SELECT p.*, c.name AS category_name
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.price < 100
  AND p.attributes @> '{"brand": "Apple"}';
```

The relational WHERE clause uses B-tree indexes, and the JSONB containment uses GIN indexes. Both work together efficiently.

## Common Pitfalls
1. **Storing everything in JSONB** — Overusing JSONB leads to lost referential integrity, no type safety, and poor query performance for core fields.
2. **Storing everything in relational columns** — Adding dozens of nullable columns for variable attributes creates wide, sparse tables that are hard to maintain.
3. **Not adding constraints on JSONB** — Without CHECK constraints, JSONB columns can contain any structure, leading to data inconsistency.

## Best Practices
1. **Put structured, constrained data in columns** — Anything with a foreign key, unique constraint, or that participates in joins should be a column.
2. **Put flexible, variable data in JSONB** — Attributes that vary between records and do not need relational features belong in JSONB.
3. **Always add a CHECK constraint on JSONB** — At minimum, enforce `jsonb_typeof(column) = 'object'` to prevent invalid data.

## Summary
- Hybrid schemas combine relational columns for core data with JSONB for flexible attributes.
- Use relational columns for fields needing constraints, joins, and aggregations.
- Use JSONB for variable-schema data, sparse columns, and external API payloads.
- Add CHECK constraints and GIN indexes on JSONB columns.
- The hybrid pattern gives you the best of both relational and document models.

## Code Examples

**Hybrid Schema Design**

```sql
-- Users table with hybrid design
CREATE TABLE users (
    -- Relational: Core identity
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now(),
    
    -- Document: Flexible profile data
    profile JSONB DEFAULT '{}',
    
    -- Document: User preferences
    settings JSONB DEFAULT '{
        "theme": "light",
        "notifications": true,
        "language": "en"
    }'
);

-- Query combining both
SELECT 
    id, email,
    profile->>'display_name' AS name,
    settings->>'theme' AS theme
FROM users
WHERE email LIKE '%@company.com'
  AND settings @> '{"notifications": true}';
```


## Resources

- [JSON Types](https://www.postgresql.org/docs/current/datatype-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*