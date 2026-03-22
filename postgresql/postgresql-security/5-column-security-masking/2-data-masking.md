---
source_course: "postgresql-security"
source_lesson: "postgresql-security-data-masking"
---

# Dynamic Data Masking with Views

## Introduction
Data masking hides or transforms sensitive data while preserving data utility. In PostgreSQL, views are the primary mechanism for dynamic masking: you create a view that applies masking functions, then grant access to the view instead of the underlying table. This lets different roles see different levels of detail from the same data.

## Key Concepts
- **Masking View**: A view that applies transformation functions (redaction, partial masking) to sensitive columns.
- **Role-Based Masking**: Using `current_user` or `pg_has_role()` in CASE expressions to show different levels of detail per role.
- **security_barrier**: A view option that prevents the query optimizer from reordering predicates in ways that could leak masked data.

## Real World Context
A customer support portal needs to display customer information, but support agents should only see masked credit card numbers (last 4 digits) and masked emails (domain only). Creating a masked view lets you provide useful data without exposing PII, satisfying both usability and compliance requirements.

## Deep Dive

### Basic View-Based Masking

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    phone TEXT,
    credit_card TEXT,
    ssn TEXT
);

CREATE VIEW customers_masked AS
SELECT
    id,
    name,
    CASE
        WHEN current_user = 'admin' THEN email
        ELSE '***@' || split_part(email, '@', 2)
    END AS email,
    CASE
        WHEN current_user IN ('admin', 'support') THEN phone
        ELSE '***-***-' || right(phone, 4)
    END AS phone,
    '****-****-****-' || right(credit_card, 4) AS credit_card,
    '***-**-' || right(ssn, 4) AS ssn
FROM customers;

REVOKE ALL ON customers FROM PUBLIC;
GRANT SELECT ON customers_masked TO app_user;
```

### Reusable Masking Functions

```sql
CREATE OR REPLACE FUNCTION mask_email(email TEXT)
RETURNS TEXT AS $$
BEGIN
    IF email IS NULL THEN RETURN NULL; END IF;
    RETURN '***@' || split_part(email, '@', 2);
END;
$$ LANGUAGE plpgsql IMMUTABLE;

CREATE OR REPLACE FUNCTION mask_credit_card(cc TEXT)
RETURNS TEXT AS $$
BEGIN
    IF cc IS NULL OR length(cc) < 4 THEN RETURN '****'; END IF;
    RETURN '****-****-****-' || right(regexp_replace(cc, '[^0-9]', '', 'g'), 4);
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

### Security Barrier Views

```sql
CREATE VIEW secure_data
WITH (security_barrier = true) AS
SELECT id, mask_email(email) AS email
FROM customers
WHERE active = true;
```

The `security_barrier` option prevents the optimizer from pushing user-supplied predicates below the view's own filters, which could otherwise leak data through timing attacks or error messages.

## Common Pitfalls
1. **Not using security_barrier** — Without it, the optimizer might evaluate a user's WHERE clause before the view's masking, potentially leaking unmasked values through errors or side channels.
2. **Granting access to the base table** — If users can access both the masked view and the base table, the masking is useless. Always REVOKE access to the base table.

## Best Practices
1. **Always use security_barrier on masking views** — This prevents optimizer-based information leakage.
2. **Create reusable masking functions** — Standardize masking logic in functions so it is consistent across all views.

## Summary
- Use views with CASE expressions to dynamically mask sensitive data per role.
- security_barrier prevents optimizer-based data leakage through predicate reordering.
- Always revoke direct access to base tables when using masking views.

## Code Examples

**Creating a security barrier view with masked email and credit card columns**

```sql
-- Create a security barrier masking view
CREATE VIEW customers_masked
WITH (security_barrier = true) AS
SELECT
    id, name,
    '***@' || split_part(email, '@', 2) AS email,
    '****-****-****-' || right(credit_card, 4) AS credit_card
FROM customers;

REVOKE ALL ON customers FROM PUBLIC;
GRANT SELECT ON customers_masked TO support_team;
```


## Resources

- [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html) — View creation including security options

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*