---
source_course: "postgresql-security"
source_lesson: "postgresql-security-schema-migration"
---

# Schema Migration Strategies

## Introduction
In a schema-per-tenant architecture, applying database migrations is more complex than in a single-schema setup. When you add a column or create an index, that change must be applied to every tenant schema. This lesson covers strategies for reliable, scalable migrations across hundreds or thousands of tenant schemas.

## Key Concepts
- **Iterative Migration**: A function that loops through all tenant schemas and applies the same DDL to each one.
- **Migration Tracking**: Recording which migrations have been applied to which schemas, so failed partial migrations can be resumed.
- **Concurrent Index Creation**: Using `CREATE INDEX CONCURRENTLY` to avoid locking tables during migrations.

## Real World Context
Your SaaS platform has 500 tenant schemas and you need to add a `phone` column to the `users` table in every schema. Running 500 ALTER TABLE statements manually is not an option. You need an automated, resumable migration process that handles failures gracefully — because schema 347 might fail due to a constraint conflict while the other 499 succeed.

## Deep Dive

### Iterative Migration Function

The core pattern is a function that iterates over tenant schemas and applies DDL:

```sql
CREATE OR REPLACE FUNCTION migrate_all_tenants(migration_sql TEXT)
RETURNS TABLE(schema_name TEXT, success BOOLEAN, error_message TEXT) AS $$
DECLARE
    s TEXT;
BEGIN
    FOR s IN
        SELECT nspname FROM pg_namespace WHERE nspname LIKE 'tenant_%'
    LOOP
        BEGIN
            EXECUTE format('SET LOCAL search_path TO %I, public', s);
            EXECUTE migration_sql;
            schema_name := s;
            success := true;
            error_message := NULL;
            RETURN NEXT;
        EXCEPTION WHEN OTHERS THEN
            schema_name := s;
            success := false;
            error_message := SQLERRM;
            RETURN NEXT;
        END;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

Usage:

```sql
SELECT * FROM migrate_all_tenants(
    'ALTER TABLE users ADD COLUMN IF NOT EXISTS phone TEXT'
);
```

### Migration Tracking Table

```sql
CREATE TABLE public.schema_migrations (
    id SERIAL PRIMARY KEY,
    migration_name TEXT NOT NULL,
    schema_name TEXT NOT NULL,
    applied_at TIMESTAMPTZ DEFAULT NOW(),
    success BOOLEAN NOT NULL,
    error_message TEXT,
    UNIQUE(migration_name, schema_name)
);
```

Before applying a migration, check if it has already been applied to each schema. This makes the process idempotent and resumable.

### Handling Long-Running Migrations

For large tables, use non-blocking techniques:

```sql
-- Add column with default (non-blocking in PG 11+)
ALTER TABLE users ADD COLUMN phone TEXT DEFAULT '';

-- Create index concurrently (cannot be inside a transaction)
CREATE INDEX CONCURRENTLY idx_users_phone ON users(phone);
```

Note: `CREATE INDEX CONCURRENTLY` cannot run inside a transaction block, so iterative migration functions need special handling for index creation.

## Common Pitfalls
1. **Running migrations in a single transaction** — If one schema fails, the entire transaction rolls back, undoing work on all other schemas. Use per-schema transactions instead.
2. **Forgetting IF NOT EXISTS / IF EXISTS** — When resuming a failed migration, DDL statements must be idempotent to avoid errors on already-migrated schemas.

## Best Practices
1. **Make all migration DDL idempotent** — Use `IF NOT EXISTS`, `IF EXISTS`, and `ADD COLUMN IF NOT EXISTS` so migrations can be safely re-run.
2. **Track migrations per schema** — Maintain a migration tracking table so you can resume partial failures and audit migration status.

## Summary
- Multi-tenant schema migrations require iterating over all tenant schemas.
- Track migration status per schema to enable resumable, auditable migrations.
- Use idempotent DDL and concurrent operations for reliability at scale.

## Code Examples

**Applying and tracking a migration across all tenant schemas**

```sql
-- Apply migration to all tenant schemas with error handling
SELECT * FROM migrate_all_tenants(
    'ALTER TABLE users ADD COLUMN IF NOT EXISTS phone TEXT'
);

-- Check migration status
SELECT schema_name, success, error_message
FROM public.schema_migrations
WHERE migration_name = 'add_phone_column'
ORDER BY schema_name;
```


## Resources

- [ALTER TABLE](https://www.postgresql.org/docs/18/sql-altertable.html) — ALTER TABLE syntax and options

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*