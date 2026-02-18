---
source_course: "postgresql-plpgsql"
source_lesson: "postgresql-plpgsql-returning-data"
---

# Returning Data from Functions

Functions can return scalar values, rows, or entire result sets.

## Scalar Returns

```plpgsql
CREATE FUNCTION get_total_orders(user_id INTEGER) RETURNS INTEGER AS $$
DECLARE
    total INTEGER;
BEGIN
    SELECT COUNT(*) INTO total FROM orders WHERE user_id = $1;
    RETURN total;
END;
$$ LANGUAGE plpgsql;
```

## Returning a Row (Composite Type)

```plpgsql
CREATE FUNCTION get_user_details(user_id INTEGER) 
RETURNS users AS $$
DECLARE
    result users%ROWTYPE;
BEGIN
    SELECT * INTO result FROM users WHERE id = user_id;
    RETURN result;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM get_user_details(1);
SELECT (get_user_details(1)).name;  -- Access specific field
```

## SETOF: Returning Multiple Rows

### Using RETURN NEXT

```plpgsql
CREATE FUNCTION get_active_users() RETURNS SETOF users AS $$
DECLARE
    user_rec users%ROWTYPE;
BEGIN
    FOR user_rec IN SELECT * FROM users WHERE active = true LOOP
        -- Can modify data before returning
        RETURN NEXT user_rec;
    END LOOP;
    RETURN;  -- Final return
END;
$$ LANGUAGE plpgsql;

-- Usage (acts like a table)
SELECT * FROM get_active_users();
```

### Using RETURN QUERY

```plpgsql
CREATE FUNCTION search_products(search_term TEXT) 
RETURNS SETOF products AS $$
BEGIN
    RETURN QUERY 
        SELECT * FROM products 
        WHERE name ILIKE '%' || search_term || '%';
    
    -- Can have multiple RETURN QUERY statements
    RETURN QUERY
        SELECT * FROM products
        WHERE description ILIKE '%' || search_term || '%';
    
    RETURN;
END;
$$ LANGUAGE plpgsql;
```

## TABLE Return Type

Define custom output columns:

```plpgsql
CREATE FUNCTION get_order_summary(p_user_id INTEGER)
RETURNS TABLE(
    order_id INTEGER,
    total NUMERIC,
    item_count BIGINT,
    order_date DATE
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        o.id,
        o.total,
        COUNT(oi.id),
        o.created_at::DATE
    FROM orders o
    LEFT JOIN order_items oi ON o.id = oi.order_id
    WHERE o.user_id = p_user_id
    GROUP BY o.id, o.total, o.created_at;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM get_order_summary(42);
```

## OUT Parameters

Alternative way to return multiple values:

```plpgsql
CREATE FUNCTION get_stats(
    IN p_user_id INTEGER,
    OUT order_count INTEGER,
    OUT total_spent NUMERIC,
    OUT avg_order NUMERIC
) AS $$
BEGIN
    SELECT COUNT(*), SUM(total), AVG(total)
    INTO order_count, total_spent, avg_order
    FROM orders
    WHERE user_id = p_user_id;
    
    -- No explicit RETURN needed - OUT params are returned
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM get_stats(42);
SELECT (get_stats(42)).total_spent;
```

📖 [Control Structures](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-STATEMENTS-RETURNING)

---

> 📘 *This lesson is part of the [Server-Side Programming with PL/pgSQL](https://stanza.dev/courses/postgresql-plpgsql) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*