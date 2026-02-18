---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-trigger-patterns"
---

# Common Trigger Patterns

Practical trigger implementations for real applications.

## Audit Trail

```plpgsql
-- Audit table
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    operation TEXT NOT NULL,
    old_data JSONB,
    new_data JSONB,
    changed_by TEXT DEFAULT current_user,
    changed_at TIMESTAMPTZ DEFAULT NOW()
);

-- Generic audit function
CREATE OR REPLACE FUNCTION audit_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD));
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_data)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(NEW));
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Apply to table
CREATE TRIGGER users_audit
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_func();
```

## Data Validation

```plpgsql
CREATE OR REPLACE FUNCTION validate_order()
RETURNS TRIGGER AS $$
BEGIN
    -- Check total is positive
    IF NEW.total <= 0 THEN
        RAISE EXCEPTION 'Order total must be positive';
    END IF;
    
    -- Check user exists
    IF NOT EXISTS (SELECT 1 FROM users WHERE id = NEW.user_id) THEN
        RAISE EXCEPTION 'Invalid user_id: %', NEW.user_id;
    END IF;
    
    -- Auto-set created_at for new orders
    IF TG_OP = 'INSERT' THEN
        NEW.created_at := NOW();
        NEW.status := 'pending';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validate_order_trigger
    BEFORE INSERT OR UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION validate_order();
```

## Maintaining Derived Data

```plpgsql
-- Keep denormalized count in sync
CREATE OR REPLACE FUNCTION update_order_count()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE users 
        SET order_count = order_count + 1
        WHERE id = NEW.user_id;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE users 
        SET order_count = order_count - 1
        WHERE id = OLD.user_id;
    ELSIF TG_OP = 'UPDATE' AND OLD.user_id != NEW.user_id THEN
        -- Order moved to different user
        UPDATE users SET order_count = order_count - 1 WHERE id = OLD.user_id;
        UPDATE users SET order_count = order_count + 1 WHERE id = NEW.user_id;
    END IF;
    RETURN NULL;  -- AFTER trigger, return value ignored
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER sync_order_count
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_order_count();
```

## Preventing Changes

```plpgsql
CREATE OR REPLACE FUNCTION prevent_delete_admin()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.role = 'admin' THEN
        RAISE EXCEPTION 'Cannot delete admin users';
    END IF;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER no_delete_admin
    BEFORE DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION prevent_delete_admin();
```

## Conditional Triggers (WHEN clause)

```plpgsql
-- Only log significant price changes
CREATE TRIGGER log_price_change
    AFTER UPDATE OF price ON products
    FOR EACH ROW
    WHEN (OLD.price IS DISTINCT FROM NEW.price 
          AND ABS(NEW.price - OLD.price) > 10)
    EXECUTE FUNCTION log_significant_change();
```

📖 [Trigger Procedures](https://www.postgresql.org/docs/18/plpgsql-trigger.html)

## Resources

- [Trigger Functions](https://www.postgresql.org/docs/18/plpgsql-trigger.html) — Writing trigger functions

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*