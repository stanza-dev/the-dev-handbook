---
source_course: "postgresql-security"
source_lesson: "postgresql-security-rls-policies"
---

# Writing RLS Policies

Create effective row-level security policies.

## USING vs WITH CHECK

- **USING**: Filters existing rows (SELECT, UPDATE, DELETE)
- **WITH CHECK**: Validates new/modified rows (INSERT, UPDATE)

```sql
-- Users can only see their own orders
CREATE POLICY see_own_orders ON orders
    FOR SELECT
    USING (user_id = current_user_id());

-- Users can only insert orders for themselves
CREATE POLICY insert_own_orders ON orders
    FOR INSERT
    WITH CHECK (user_id = current_user_id());

-- Combined: can update own orders, must remain own orders
CREATE POLICY update_own_orders ON orders
    FOR UPDATE
    USING (user_id = current_user_id())
    WITH CHECK (user_id = current_user_id());
```

## Multi-Tenant Example

```sql
-- Tenant isolation
CREATE TABLE tenant_data (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    data TEXT
);

ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

-- Each user only sees their tenant's data
CREATE POLICY tenant_isolation ON tenant_data
    FOR ALL
    USING (tenant_id = current_setting('app.tenant_id')::INTEGER)
    WITH CHECK (tenant_id = current_setting('app.tenant_id')::INTEGER);

-- Application sets tenant context
SET app.tenant_id = '42';
SELECT * FROM tenant_data;  -- Only sees tenant 42's data
```

## Role-Specific Policies

```sql
-- Admins see everything
CREATE POLICY admin_all ON orders
    FOR ALL
    TO admin_role
    USING (true);

-- Regular users see only their orders
CREATE POLICY user_own_orders ON orders
    FOR ALL
    TO app_user
    USING (user_id = current_user_id())
    WITH CHECK (user_id = current_user_id());
```

## Hierarchical Access

```sql
-- Managers see their team's data
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    manager_id INTEGER REFERENCES employees(id),
    name TEXT,
    salary NUMERIC
);

CREATE POLICY manager_sees_team ON employees
    FOR SELECT
    USING (
        id = current_user_id()  -- See own record
        OR manager_id = current_user_id()  -- See direct reports
        OR id IN (  -- Or recursive subordinates
            WITH RECURSIVE subordinates AS (
                SELECT id FROM employees WHERE manager_id = current_user_id()
                UNION
                SELECT e.id FROM employees e
                JOIN subordinates s ON e.manager_id = s.id
            )
            SELECT id FROM subordinates
        )
    );
```

## PERMISSIVE vs RESTRICTIVE

```sql
-- PERMISSIVE (default): policies OR together
CREATE POLICY allow_own ON docs FOR SELECT
    USING (owner = current_user);  -- PERMISSIVE

CREATE POLICY allow_public ON docs FOR SELECT
    USING (is_public = true);  -- PERMISSIVE
-- Result: (owner = current_user) OR (is_public = true)

-- RESTRICTIVE: must pass along with permissive policies
CREATE POLICY active_only ON docs FOR SELECT
    AS RESTRICTIVE
    USING (is_active = true);
-- Result: ((owner = current_user) OR (is_public = true)) AND (is_active = true)
```

📖 [CREATE POLICY](https://www.postgresql.org/docs/18/sql-createpolicy.html)

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*