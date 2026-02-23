---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-raw-queries"
---

# Raw SQL Queries

While Django's ORM handles most queries, sometimes you need raw SQL for complex operations, database-specific features, or performance optimization.

## Manager.raw() - Return Model Instances

```python
# Returns model instances (Person objects)
people = Person.objects.raw('SELECT * FROM myapp_person')

for person in people:
    print(person.first_name)  # Access as normal model attributes
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### With Parameters (Safe from SQL Injection)

The following example demonstrates how to use with parameters (safe from sql injection) in practice:

```python
# ALWAYS use parameters for user input
name = "John"
people = Person.objects.raw(
    'SELECT * FROM myapp_person WHERE first_name = %s',
    [name]
)

# Named parameters
people = Person.objects.raw(
    'SELECT * FROM myapp_person WHERE first_name = %(name)s',
    {'name': 'John'}
)
```

⚠️ **Never use string formatting:**
```python
# DANGEROUS - SQL injection vulnerability!
Person.objects.raw(f"SELECT * FROM myapp_person WHERE name = '{name}'")
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Mapping Columns to Fields

The following example demonstrates how to use mapping columns to fields in practice:

```python
# When column names don't match field names
people = Person.objects.raw(
    'SELECT id, first_name AS fname FROM myapp_person',
    translations={'fname': 'first_name'}
)
```

## connection.cursor() - Execute Arbitrary SQL

For queries that don't map to models:

```python
from django.db import connection

def get_statistics():
    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT 
                DATE(created_at) as date,
                COUNT(*) as count,
                AVG(amount) as avg_amount
            FROM orders
            WHERE created_at > %s
            GROUP BY DATE(created_at)
        """, [last_month])
        
        # Fetch results
        rows = cursor.fetchall()
        # [('2024-01-01', 50, 75.50), ('2024-01-02', 45, 82.30), ...]
        
        return rows
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Fetching Methods

The following example demonstrates how to use fetching methods in practice:

```python
with connection.cursor() as cursor:
    cursor.execute('SELECT * FROM users')
    
    # Fetch all rows
    all_rows = cursor.fetchall()
    
    # Fetch one row
    one_row = cursor.fetchone()
    
    # Fetch N rows
    some_rows = cursor.fetchmany(10)
    
    # Get column names
    columns = [col[0] for col in cursor.description]
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Return Dictionaries

The following example demonstrates how to use return dictionaries in practice:

```python
def dictfetchall(cursor):
    """Return all rows from a cursor as a list of dicts."""
    columns = [col[0] for col in cursor.description]
    return [dict(zip(columns, row)) for row in cursor.fetchall()]

with connection.cursor() as cursor:
    cursor.execute('SELECT * FROM users')
    results = dictfetchall(cursor)
    # [{'id': 1, 'name': 'John'}, {'id': 2, 'name': 'Jane'}, ...]
```

## RawSQL Expression

Embed raw SQL in ORM queries:

```python
from django.db.models.expressions import RawSQL

# Annotate with raw SQL
Person.objects.annotate(
    full_name=RawSQL(
        "CONCAT(first_name, ' ', last_name)",
        []
    )
)

# Filter with raw SQL
Person.objects.filter(
    id__in=RawSQL(
        "SELECT user_id FROM active_sessions WHERE expires_at > %s",
        [timezone.now()]
    )
)
```

## When to Use Raw SQL

✅ **Good Use Cases:**
- Complex GROUP BY with HAVING
- Database-specific functions (JSON, GIS)
- Performance-critical reports
- Migrating legacy queries

❌ **Avoid When:**
- ORM can do it (even if less elegantly)
- Portability between databases matters
- Simple CRUD operations

## Database-Specific Features

```python
# PostgreSQL EXPLAIN ANALYZE
with connection.cursor() as cursor:
    cursor.execute('EXPLAIN ANALYZE SELECT * FROM large_table WHERE status = %s', ['active'])
    print(cursor.fetchall())

# PostgreSQL-specific JSON query
Person.objects.raw("""
    SELECT * FROM myapp_person
    WHERE metadata->>'role' = %s
""", ['admin'])
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test raw sql queries with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some raw sql queries features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- Raw SQL Queries is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from Raw SQL Queries**

```python
# Returns model instances (Person objects)
people = Person.objects.raw('SELECT * FROM myapp_person')

for person in people:
    print(person.first_name)  # Access as normal model attributes
```


## Resources

- [Performing Raw SQL Queries](https://docs.djangoproject.com/en/6.0/topics/db/sql/) — Official guide to raw SQL in Django

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*