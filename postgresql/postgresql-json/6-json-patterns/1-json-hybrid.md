---
source_course: "postgresql-json"
source_lesson: "relational-json-hybrid"
---

# Hybrid Relational and Document Design

PostgreSQL excels at combining relational and document models. The key is knowing when to use each approach.

## When to Use JSONB

1. **Variable schemas**: Product attributes, form responses
2. **Nested data**: Address components, metadata
3. **External API data**: Preserve original structure
4. **Sparse columns**: Only store what's needed
5. **Rapidly evolving schemas**: Avoid migrations

## When to Use Relational

1. **Frequent joins**: Data referenced by other tables
2. **Unique constraints**: Email, username
3. **Foreign keys**: Referential integrity needed
4. **Aggregations**: SUM, AVG on columns
5. **Strict validation**: Data must conform to schema

## Hybrid Pattern Example

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

## Query Patterns

```sql
-- Combine relational and JSON filtering
SELECT p.*, c.name AS category_name
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.price < 100
  AND p.attributes @> '{"brand": "Apple"}';
```

## Code Examples

```undefined
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