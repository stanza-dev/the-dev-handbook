---
source_course: "postgresql-security"
source_lesson: "postgresql-security-rls-intro"
---

# Introduction to Row-Level Security

Row-Level Security (RLS) restricts which rows users can see or modify.

## Why Row-Level Security?

**Traditional approach:**
```sql
-- Filter in every query
SELECT * FROM orders WHERE user_id = current_user_id();
-- Easy to forget, error-prone
```

**RLS approach:**
```sql
-- Policy enforces automatically
SELECT * FROM orders;  -- Only sees their own orders
```

## Use Cases

- Multi-tenant applications (each tenant sees only their data)
- User-specific data (users see only their records)
- Hierarchical access (managers see team data)
- Compliance (GDPR, HIPAA data isolation)

## Enabling RLS

```sql
-- Create table
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    owner TEXT NOT NULL,
    content TEXT,
    is_public BOOLEAN DEFAULT false
);

-- Enable RLS (without policies, denies all access!)
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Force RLS even for table owner (optional but recommended)
ALTER TABLE documents FORCE ROW LEVEL SECURITY;
```

## Creating Policies

```sql
-- Allow users to see their own documents
CREATE POLICY user_sees_own_docs ON documents
    FOR SELECT
    USING (owner = current_user);

-- Allow users to see public documents
CREATE POLICY public_docs_visible ON documents
    FOR SELECT
    USING (is_public = true);
```

## How Policies Combine

**Multiple policies for same command type:**
- Combined with OR (any policy can allow access)

```sql
-- User sees docs where:
-- (owner = current_user) OR (is_public = true)
```

## Policy Components

```sql
CREATE POLICY name ON table
    [AS {PERMISSIVE | RESTRICTIVE}]  -- How to combine
    [FOR {ALL | SELECT | INSERT | UPDATE | DELETE}]  -- Which operations
    [TO role_name]  -- Which roles (default: PUBLIC)
    [USING (expression)]  -- Filter existing rows
    [WITH CHECK (expression)];  -- Validate new/modified rows
```

📖 [Row Security Policies](https://www.postgresql.org/docs/18/ddl-rowsecurity.html)

## Resources

- [Row Security Policies](https://www.postgresql.org/docs/18/ddl-rowsecurity.html) — Complete RLS documentation

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*