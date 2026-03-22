---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-event-triggers"
---

# Event Triggers

## Introduction

While regular triggers respond to data changes (DML), event triggers respond to schema changes (DDL). They fire when someone creates, alters, or drops database objects. Event triggers are essential for schema governance, change auditing, and preventing accidental destructive operations in production databases.

## Key Concepts

- **Event Trigger**: A trigger that fires on DDL commands (CREATE, ALTER, DROP, etc.) rather than data changes.
- **ddl_command_start / ddl_command_end**: Events fired before and after a DDL command executes.
- **sql_drop**: Fired after objects are dropped, with access to the list of dropped objects.
- **table_rewrite**: Fired before a table is rewritten (e.g., ALTER TABLE that changes column types).
- **TAG filter**: Limits which DDL commands fire the trigger (e.g., only CREATE TABLE and DROP TABLE).

## Real World Context

A production database should never have tables dropped by accident. An event trigger on `sql_drop` can prevent all table drops in the `public` schema, forcing developers to go through a controlled migration process. A separate trigger on `ddl_command_end` logs all schema changes for compliance and debugging.

## Deep Dive

### Basic Event Trigger

```sql
CREATE OR REPLACE FUNCTION log_ddl_command()
RETURNS event_trigger AS $$
DECLARE
    obj RECORD;
BEGIN
    FOR obj IN SELECT * FROM pg_event_trigger_ddl_commands() LOOP
        INSERT INTO ddl_log (command_tag, object_type, schema_name, object_identity)
        VALUES (obj.command_tag, obj.object_type, obj.schema_name, obj.object_identity);
    END LOOP;
END;
$$ LANGUAGE plpgsql;

CREATE EVENT TRIGGER log_ddl
    ON ddl_command_end
    EXECUTE FUNCTION log_ddl_command();
```

### Preventing Schema Changes

```sql
CREATE OR REPLACE FUNCTION prevent_drop_table()
RETURNS event_trigger AS $$
DECLARE
    obj RECORD;
BEGIN
    FOR obj IN SELECT * FROM pg_event_trigger_dropped_objects() LOOP
        IF obj.object_type = 'table' AND obj.schema_name = 'public' THEN
            RAISE EXCEPTION 'Dropping tables in public schema is not allowed!';
        END IF;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

CREATE EVENT TRIGGER no_drop_public_tables
    ON sql_drop
    EXECUTE FUNCTION prevent_drop_table();
```

### Filtering Events

```sql
CREATE EVENT TRIGGER audit_table_changes
    ON ddl_command_end
    WHEN TAG IN ('CREATE TABLE', 'ALTER TABLE', 'DROP TABLE')
    EXECUTE FUNCTION audit_table_ddl();
```

### Event Trigger Special Variables

- `tg_event`: The event name (e.g., 'ddl_command_end')
- `tg_tag`: The command tag (e.g., 'CREATE TABLE')

### Disabling Event Triggers

```sql
ALTER EVENT TRIGGER log_ddl DISABLE;
-- Perform maintenance...
ALTER EVENT TRIGGER log_ddl ENABLE;
```

## Common Pitfalls

1. **Event triggers blocking migrations** -- A trigger that prevents DROP TABLE will also block migration tools. Use ALTER EVENT TRIGGER DISABLE before running migrations.
2. **Event triggers cannot fire in a transaction that creates them** -- The trigger is not active until the transaction that creates it commits.

## Best Practices

1. **Use TAG filters to narrow scope** -- A ddl_command_end trigger without filters fires on every DDL statement, including harmless ones like CREATE INDEX.
2. **Provide a documented override mechanism** -- Document how to temporarily disable protective event triggers for legitimate maintenance.

## Summary

- Event triggers respond to DDL commands (schema changes) rather than data changes.
- Use ddl_command_end with TAG filters for change auditing and sql_drop for preventing accidental destruction.
- Always provide a documented way to disable protective event triggers during legitimate maintenance operations.

## Code Examples

**An event trigger that blocks accidental table drops in the public schema, useful for production database protection.**

```sql
CREATE OR REPLACE FUNCTION prevent_drop_table()
RETURNS event_trigger AS $$
DECLARE
    obj RECORD;
BEGIN
    FOR obj IN SELECT * FROM pg_event_trigger_dropped_objects() LOOP
        IF obj.object_type = 'table' AND obj.schema_name = 'public' THEN
            RAISE EXCEPTION 'Dropping tables in public schema is not allowed!';
        END IF;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

CREATE EVENT TRIGGER no_drop_public_tables
    ON sql_drop
    EXECUTE FUNCTION prevent_drop_table();
```


## Resources

- [Event Triggers](https://www.postgresql.org/docs/18/event-triggers.html) — DDL event triggers

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*