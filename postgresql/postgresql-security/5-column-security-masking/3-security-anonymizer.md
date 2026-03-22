---
source_course: "postgresql-security"
source_lesson: "postgresql-security-anonymizer"
---

# PostgreSQL Anonymizer Extension

## Introduction
While views and custom functions provide basic masking, the PostgreSQL Anonymizer extension (`anon`) offers a declarative, label-based approach to data masking. You annotate columns with masking rules using security labels, and the extension automatically applies them. This is especially valuable for GDPR compliance and large schemas where manually creating masking views for every table is impractical.

## Key Concepts
- **Security Labels**: PostgreSQL's built-in mechanism for attaching metadata to database objects, used by the Anonymizer to define masking rules.
- **Masking Rules**: Declarative annotations specifying how to transform a column (e.g., fake name, partial masking, randomization).
- **Static vs Dynamic Masking**: Static masking permanently replaces data in-place; dynamic masking applies rules at query time per role.

## Real World Context
A company with 200 tables containing PII needs to provide developers with realistic test data. Instead of writing 200 masking views, they install the Anonymizer extension, label sensitive columns with masking rules, and generate a masked dump. Developers get realistic data without any PII, and the masking rules are version-controlled alongside the schema.

## Deep Dive

### Installing and Configuring

```sql
CREATE EXTENSION anon CASCADE;
SELECT anon.init();
```

### Declaring Masking Rules

Rules are declared using security labels:

```sql
-- Mask with a fake name
SECURITY LABEL FOR anon ON COLUMN customers.name
    IS 'MASKED WITH FUNCTION anon.fake_last_name()';

-- Partial masking for email
SECURITY LABEL FOR anon ON COLUMN customers.email
    IS 'MASKED WITH FUNCTION anon.partial_email(email)';

-- Random phone number
SECURITY LABEL FOR anon ON COLUMN customers.phone
    IS 'MASKED WITH FUNCTION anon.random_phone()';

-- Mask credit card
SECURITY LABEL FOR anon ON COLUMN customers.credit_card
    IS 'MASKED WITH VALUE ''****-****-****-****''';
```

### Dynamic Masking

Dynamic masking applies rules in real-time based on the querying role:

```sql
-- Enable dynamic masking
SELECT anon.start_dynamic_masking();

-- Declare a masked role
SECURITY LABEL FOR anon ON ROLE analyst
    IS 'MASKED';

-- Now when analyst queries, they see masked data
SET ROLE analyst;
SELECT * FROM customers;
-- name: 'Smith', email: 'a]***@example.com', credit_card: '****-****-****-****'
```

### Static Masking (Anonymization)

```sql
-- Permanently replace data (irreversible!)
SELECT anon.anonymize_database();
```

This is useful for creating sanitized copies of production data for development or testing.

### SECURITY DEFINER Functions for Custom Masking

For complex masking logic that needs elevated privileges, use SECURITY DEFINER:

```sql
CREATE OR REPLACE FUNCTION mask_salary_range(salary NUMERIC)
RETURNS TEXT
SECURITY DEFINER
AS $$
BEGIN
    RETURN CASE
        WHEN salary < 50000 THEN '< 50K'
        WHEN salary < 100000 THEN '50K-100K'
        WHEN salary < 150000 THEN '100K-150K'
        ELSE '150K+'
    END;
END;
$$ LANGUAGE plpgsql;

-- The function runs with the definer's privileges
-- even when called by a low-privilege role
```

## Common Pitfalls
1. **Running static anonymization on production** — `anon.anonymize_database()` permanently destroys original data. Only use on copies.
2. **Forgetting to init** — The extension requires `SELECT anon.init()` before masking rules take effect.

## Best Practices
1. **Use dynamic masking for development access** — Let developers query production-like data without seeing real PII.
2. **Version-control masking rules** — Include SECURITY LABEL statements in your migration scripts so masking rules evolve with the schema.

## Summary
- The PostgreSQL Anonymizer extension provides declarative, label-based data masking.
- Dynamic masking applies rules per-role at query time; static masking permanently replaces data.
- SECURITY DEFINER functions enable custom masking logic with elevated privileges.

## Code Examples

**Setting up the PostgreSQL Anonymizer extension with dynamic masking for a role**

```sql
-- Set up declarative masking with the anon extension
CREATE EXTENSION anon CASCADE;
SELECT anon.init();

SECURITY LABEL FOR anon ON COLUMN customers.email
    IS 'MASKED WITH FUNCTION anon.partial_email(email)';

SECURITY LABEL FOR anon ON ROLE analyst IS 'MASKED';
SELECT anon.start_dynamic_masking();
```


## Resources

- [PostgreSQL Anonymizer](https://postgresql-anonymizer.readthedocs.io/en/stable/) — Data masking and anonymization extension

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*