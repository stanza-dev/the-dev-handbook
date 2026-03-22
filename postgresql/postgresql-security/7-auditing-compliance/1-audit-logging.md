---
source_course: "postgresql-security"
source_lesson: "postgresql-security-audit-logging"
---

# Audit Logging

## Introduction
Audit logging tracks who did what and when in your database. It is essential for security incident investigation, compliance requirements (GDPR, HIPAA, PCI-DSS, SOC 2), and operational debugging. PostgreSQL provides built-in logging, the pgAudit extension for structured audit trails, and trigger-based approaches for change tracking.

## Key Concepts
- **log_statement**: Built-in PostgreSQL setting that controls which SQL statements are logged (none, ddl, mod, all).
- **pgAudit**: An extension that provides detailed, structured audit logging with fine-grained control over what is logged.
- **Trigger-Based Audit**: Application-level change tracking using AFTER triggers that record old and new values.

## Real World Context
During a security incident, you need to answer: "Who accessed the customers table in the last 24 hours, and what did they read or modify?" Built-in logging with `log_statement = 'all'` captures all SQL. pgAudit provides structured, parseable logs. A trigger-based audit trail on the customers table records the exact old and new values for every change.

## Deep Dive

### Built-in Logging

```ini
# postgresql.conf
log_statement = 'all'
log_connections = on
log_disconnections = on
log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a '
log_duration = on
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d.log'
```

### log_statement Options

| Value | Logs |
|-------|------|
| `none` | Nothing |
| `ddl` | CREATE, ALTER, DROP |
| `mod` | DDL + INSERT, UPDATE, DELETE |
| `all` | All statements |

### pgAudit Extension

```sql
CREATE EXTENSION pgaudit;
```

```ini
# postgresql.conf
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'write, ddl, role'
pgaudit.log_catalog = off
pgaudit.log_parameter = on
```

### pgAudit Log Classes

| Class | Logs |
|-------|------|
| `READ` | SELECT, COPY |
| `WRITE` | INSERT, UPDATE, DELETE, TRUNCATE |
| `FUNCTION` | Function calls |
| `ROLE` | GRANT, REVOKE, CREATE/ALTER/DROP ROLE |
| `DDL` | All DDL not in ROLE |
| `MISC` | DISCARD, FETCH, CHECKPOINT, etc. |

### Trigger-Based Audit Trail

```sql
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL,
    user_name TEXT NOT NULL DEFAULT current_user,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    old_data JSONB,
    new_data JSONB
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_data, new_data)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN to_jsonb(OLD) END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) END
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_users
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger();
```

## Common Pitfalls
1. **Setting log_statement = 'all' on high-traffic databases** — This generates enormous log files and can impact performance. Use pgAudit with targeted log classes instead.
2. **Not rotating log files** — Without log rotation, log files grow unbounded and fill the disk, potentially causing database crashes.

## Best Practices
1. **Use pgAudit for compliance-grade logging** — It provides structured, filterable audit logs suitable for compliance requirements.
2. **Use trigger-based audit for change tracking** — When you need to see the exact before/after values of data changes, triggers provide row-level detail that log-based auditing cannot.

## Summary
- PostgreSQL offers three levels of auditing: built-in logging, pgAudit extension, and trigger-based change tracking.
- Use pgAudit for compliance requirements (HIPAA, PCI-DSS) with targeted log classes.
- Trigger-based auditing captures exact old/new values for sensitive tables.

## Code Examples

**Setting up a trigger-based audit trail for tracking all changes to a table**

```sql
-- Create generic audit trigger
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL,
    user_name TEXT DEFAULT current_user,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    old_data JSONB, new_data JSONB
);

CREATE TRIGGER audit_orders
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW EXECUTE FUNCTION audit_trigger();
```


## Resources

- [Logging Configuration](https://www.postgresql.org/docs/18/runtime-config-logging.html) — Server logging configuration

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*