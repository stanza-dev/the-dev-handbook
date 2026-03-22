---
source_course: "postgresql-security"
source_lesson: "postgresql-security-column-privileges"
---

# Column-Level Privileges

## Introduction
PostgreSQL allows granting privileges on individual columns, providing fine-grained access control beyond the table level. This is essential when a table contains both public and sensitive data — such as an `employees` table with name, email, salary, and SSN columns — and different roles need access to different subsets of columns.

## Key Concepts
- **Column-Level GRANT**: Restricts SELECT, INSERT, or UPDATE to specific columns rather than the entire table.
- **Implicit Denial**: Columns not explicitly granted are inaccessible — `SELECT *` will fail if any column is denied.
- **Combining with RLS**: Column privileges control which columns; RLS controls which rows. Together they provide two-dimensional access control.

## Real World Context
An HR system has an `employees` table. The support team needs name and email to handle tickets, but should never see salary or SSN. Rather than creating separate tables or views, you grant `SELECT (id, name, email)` to the support role. Any attempt to query salary or SSN fails immediately at the database level.

## Deep Dive

### Granting Column Access

```sql
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
```

### Testing Column Access

```sql
SET ROLE hr_viewer;

-- This works
SELECT id, name, department FROM employees;

-- This fails: permission denied for column salary
SELECT salary FROM employees;

-- This also fails (salary is included in *)
SELECT * FROM employees;
```

### Column-Level INSERT/UPDATE

```sql
GRANT UPDATE (email, department) ON employees TO hr_editor;

SET ROLE hr_editor;
UPDATE employees SET email = 'new@example.com' WHERE id = 1;  -- Works
UPDATE employees SET salary = 100000 WHERE id = 1;            -- Fails
```

### Combining with Row-Level Security

```sql
-- Column privileges control WHICH columns
-- RLS controls WHICH rows

CREATE POLICY manager_access ON employees
    FOR SELECT TO manager_role
    USING (department = current_setting('app.department'));

GRANT SELECT ON employees TO manager_role;
GRANT SELECT (id, name, department) ON employees TO viewer_role;
```

### Checking Column Permissions

```sql
SELECT grantee, table_name, column_name, privilege_type
FROM information_schema.column_privileges
WHERE table_name = 'employees'
ORDER BY grantee, column_name;
```

## Common Pitfalls
1. **Forgetting that SELECT * includes all columns** — If a role has column-level grants, `SELECT *` will fail because it tries to read denied columns. Always use explicit column lists.
2. **Mixing table-level and column-level grants** — A table-level GRANT SELECT overrides column restrictions. Be consistent in your grant strategy.

## Best Practices
1. **Use column-level grants for sensitive data** — Grant access to specific columns rather than entire tables when the table contains mixed-sensitivity data.
2. **Document your column access matrix** — Maintain a table showing which roles can access which columns to avoid privilege drift.

## Summary
- Column-level privileges restrict access to specific columns within a table.
- SELECT * fails if any column is denied — always use explicit column lists.
- Combine column privileges with RLS for two-dimensional access control.

## Code Examples

**Setting up and verifying column-level privileges on an employees table**

```sql
-- Grant column-level access
GRANT SELECT (id, name, email, department) ON employees TO hr_viewer;
GRANT UPDATE (email, department) ON employees TO hr_editor;

-- Check column permissions
SELECT grantee, column_name, privilege_type
FROM information_schema.column_privileges
WHERE table_name = 'employees'
ORDER BY grantee, column_name;
```


## Resources

- [GRANT Command](https://www.postgresql.org/docs/18/sql-grant.html) — Complete GRANT syntax including column-level

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*