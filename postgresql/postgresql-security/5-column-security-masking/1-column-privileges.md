---
source_course: "postgresql-security"
source_lesson: "postgresql-security-column-privileges"
---

# Column-Level Privileges

PostgreSQL allows granting privileges on individual columns, providing fine-grained access control.

## Granting Column Access

```sql
-- Table with sensitive data
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    salary NUMERIC(10,2),
    ssn TEXT,
    department TEXT
);

-- Grant SELECT only on non-sensitive columns
GRANT SELECT (id, name, email, department) ON employees TO hr_viewer;

-- Deny access to sensitive columns
-- No explicit grant on salary or ssn
```

## Testing Column Access

```sql
-- As hr_viewer
SET ROLE hr_viewer;

-- This works
SELECT id, name, department FROM employees;

-- This fails: permission denied for column salary
SELECT salary FROM employees;

-- This also fails
SELECT * FROM employees;
```

## Column-Level INSERT/UPDATE

```sql
-- Allow updating only specific columns
GRANT UPDATE (email, department) ON employees TO hr_editor;

-- As hr_editor
SET ROLE hr_editor;

-- Works
UPDATE employees SET email = 'new@example.com' WHERE id = 1;

-- Fails
UPDATE employees SET salary = 100000 WHERE id = 1;
```

## Combining with Row-Level Security

```sql
-- Column privileges control WHICH columns
-- RLS controls WHICH rows

-- Manager can see all columns for their team
CREATE POLICY manager_access ON employees
    FOR SELECT
    TO manager_role
    USING (department = current_setting('app.department'));

-- Grant all columns to managers
GRANT SELECT ON employees TO manager_role;

-- Viewer sees only non-sensitive columns, all rows
GRANT SELECT (id, name, department) ON employees TO viewer_role;
```

## Checking Column Permissions

```sql
SELECT 
    grantee,
    table_name,
    column_name,
    privilege_type
FROM information_schema.column_privileges
WHERE table_name = 'employees'
ORDER BY grantee, column_name;
```

📖 [GRANT](https://www.postgresql.org/docs/18/sql-grant.html)

## Resources

- [GRANT Command](https://www.postgresql.org/docs/18/sql-grant.html) — Complete GRANT syntax including column-level

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*