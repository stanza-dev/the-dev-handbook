---
source_course: "postgresql-security"
source_lesson: "postgresql-security-audit-logging"
---

# Audit Logging

Track who did what and when in your database.

## Built-in Logging

```ini
# postgresql.conf

# What to log
log_statement = 'all'  # none, ddl, mod, all
log_connections = on
log_disconnections = on

# Log details
log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a '
log_duration = on

# Where to log
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d.log'
log_rotation_age = 1d
log_rotation_size = 100MB
```

### log_statement Options

| Value | Logs |
|-------|------|
| `none` | Nothing |
| `ddl` | CREATE, ALTER, DROP |
| `mod` | DDL + INSERT, UPDATE, DELETE |
| `all` | All statements |

## pgAudit Extension

More detailed, structured audit logging:

```sql
-- Install
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

## Trigger-Based Audit Trail

```sql
-- Audit table
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL,
    user_name TEXT NOT NULL DEFAULT current_user,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    old_data JSONB,
    new_data JSONB
);

-- Generic audit function
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

-- Apply to sensitive tables
CREATE TRIGGER audit_users
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger();
```

## Viewing Audit Logs

```sql
-- Recent changes to users table
SELECT 
    timestamp,
    operation,
    user_name,
    old_data->>'email' AS old_email,
    new_data->>'email' AS new_email
FROM audit_log
WHERE table_name = 'users'
  AND timestamp > NOW() - INTERVAL '24 hours'
ORDER BY timestamp DESC;
```

📖 [Error Reporting and Logging](https://www.postgresql.org/docs/18/runtime-config-logging.html)

## Resources

- [Logging Configuration](https://www.postgresql.org/docs/18/runtime-config-logging.html) — Server logging configuration

---

> 📘 *This lesson is part of the [PostgreSQL Security & Access Control](https://stanza.dev/courses/postgresql-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*