---
source_course: "django-performance"
source_lesson: "django-performance-database-connection-pooling"
---

# Database Connection Pooling

## Introduction

Connection pooling reuses database connections instead of creating new ones for each request, significantly improving performance.

## Key Concepts

**Connection Pool**: Cache of database connections.

**CONN_MAX_AGE**: Django's built-in connection persistence.

## Deep Dive

### Django's CONN_MAX_AGE

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'CONN_MAX_AGE': 60,  # Reuse connections for 60 seconds
        # 0 = close after each request (default)
        # None = unlimited persistence
    }
}
```

### Using django-db-connection-pool

```python
# pip install django-db-connection-pool

DATABASES = {
    'default': {
        'ENGINE': 'dj_db_conn_pool.backends.postgresql',
        'NAME': 'mydb',
        'POOL_OPTIONS': {
            'POOL_SIZE': 10,
            'MAX_OVERFLOW': 10,
            'RECYCLE': 300,
        }
    }
}
```

### PgBouncer (External Pooler)

```ini
# pgbouncer.ini
[databases]
mydb = host=localhost dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
```

## Best Practices

1. **Use CONN_MAX_AGE > 0 in production**: Reduces connection overhead.
2. **Match pool size to workers**: Gunicorn workers × connections per worker.
3. **Use transaction pooling for Django**: Compatible with ORM.

## Summary

Connection pooling improves performance by reusing database connections. Start with CONN_MAX_AGE, consider PgBouncer for larger deployments.

## Resources

- [Database Connections](https://docs.djangoproject.com/en/6.0/ref/databases/#connection-management) — Django connection management

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*