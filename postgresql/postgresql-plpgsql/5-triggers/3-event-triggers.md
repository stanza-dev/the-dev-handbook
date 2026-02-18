---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-event-triggers"
---

# Event Triggers

Event triggers respond to DDL (Data Definition Language) commands.

## Event Trigger Events

| Event | Description |
|-------|-------------|
| `ddl_command_start` | Before DDL command executes |
| `ddl_command_end` | After DDL command completes |
| `sql_drop` | After objects are dropped |
| `table_rewrite` | Before table is rewritten |

## Basic Event Trigger

```plpgsql
-- Log all DDL commands
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

## Preventing Schema Changes

```plpgsql
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

## Filtering Events

```plpgsql
-- Only trigger on specific commands
CREATE EVENT TRIGGER audit_table_changes
    ON ddl_command_end
    WHEN TAG IN ('CREATE TABLE', 'ALTER TABLE', 'DROP TABLE')
    EXECUTE FUNCTION audit_table_ddl();
```

## Event Trigger Special Variables

```plpgsql
CREATE OR REPLACE FUNCTION show_event_info()
RETURNS event_trigger AS $$
BEGIN
    RAISE NOTICE 'Event: %, Tag: %', tg_event, tg_tag;
END;
$$ LANGUAGE plpgsql;
```

- `tg_event`: The event name (e.g., 'ddl_command_end')
- `tg_tag`: The command tag (e.g., 'CREATE TABLE')

## Disabling Event Triggers

```plpgsql
-- Temporarily disable during maintenance
ALTER EVENT TRIGGER log_ddl DISABLE;

-- Perform maintenance...

ALTER EVENT TRIGGER log_ddl ENABLE;
```

📖 [Event Triggers](https://www.postgresql.org/docs/18/event-triggers.html)

## Resources

- [Event Triggers](https://www.postgresql.org/docs/18/event-triggers.html) — DDL event triggers

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*