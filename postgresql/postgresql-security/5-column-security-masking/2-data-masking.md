---
source_course: "postgresql-security"
source_lesson: "postgresql-security-data-masking"
---

# Dynamic Data Masking

Data masking hides or transforms sensitive data while preserving data utility.

## View-Based Masking

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    phone TEXT,
    credit_card TEXT,
    ssn TEXT
);

-- Create masked view
CREATE VIEW customers_masked AS
SELECT 
    id,
    name,
    -- Mask email: show domain only
    CASE 
        WHEN current_user = 'admin' THEN email
        ELSE '***@' || split_part(email, '@', 2)
    END AS email,
    -- Mask phone: show last 4 digits
    CASE
        WHEN current_user IN ('admin', 'support') THEN phone
        ELSE '***-***-' || right(phone, 4)
    END AS phone,
    -- Mask credit card: show last 4 digits
    '****-****-****-' || right(credit_card, 4) AS credit_card,
    -- Never show SSN
    '***-**-' || right(ssn, 4) AS ssn
FROM customers;

-- Grant access to view, not table
REVOKE ALL ON customers FROM PUBLIC;
GRANT SELECT ON customers_masked TO app_user;
```

## Role-Based Masking

```sql
-- Different masking levels by role
CREATE OR REPLACE VIEW employees_dynamic AS
SELECT 
    id,
    name,
    department,
    -- Salary visible to HR and managers
    CASE 
        WHEN pg_has_role(current_user, 'hr_role', 'MEMBER') 
             OR pg_has_role(current_user, 'manager_role', 'MEMBER')
        THEN salary::text
        ELSE 'CONFIDENTIAL'
    END AS salary,
    -- Email masking by sensitivity level
    CASE 
        WHEN pg_has_role(current_user, 'admin_role', 'MEMBER') THEN email
        WHEN pg_has_role(current_user, 'internal_role', 'MEMBER') 
            THEN split_part(email, '@', 1) || '@***'
        ELSE '***'
    END AS email
FROM employees;
```

## Function-Based Masking

```sql
-- Reusable masking functions
CREATE OR REPLACE FUNCTION mask_email(email TEXT)
RETURNS TEXT AS $$
BEGIN
    IF email IS NULL THEN RETURN NULL; END IF;
    RETURN '***@' || split_part(email, '@', 2);
END;
$$ LANGUAGE plpgsql IMMUTABLE;

CREATE OR REPLACE FUNCTION mask_phone(phone TEXT)
RETURNS TEXT AS $$
BEGIN
    IF phone IS NULL OR length(phone) < 4 THEN RETURN '****'; END IF;
    RETURN '***-***-' || right(regexp_replace(phone, '[^0-9]', '', 'g'), 4);
END;
$$ LANGUAGE plpgsql IMMUTABLE;

CREATE OR REPLACE FUNCTION mask_credit_card(cc TEXT)
RETURNS TEXT AS $$
BEGIN
    IF cc IS NULL OR length(cc) < 4 THEN RETURN '****'; END IF;
    RETURN '****-****-****-' || right(regexp_replace(cc, '[^0-9]', '', 'g'), 4);
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Use in views
CREATE VIEW payments_masked AS
SELECT 
    id,
    amount,
    mask_credit_card(card_number) AS card_number,
    status
FROM payments;
```

## Security Definer Views

```sql
-- View runs with creator's permissions
CREATE VIEW secure_data 
WITH (security_barrier = true) AS
SELECT 
    id,
    mask_email(email) AS email
FROM customers
WHERE active = true;

-- security_barrier prevents optimization leaks
```

📖 [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html)

## Resources

- [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html) — View creation including security options

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*