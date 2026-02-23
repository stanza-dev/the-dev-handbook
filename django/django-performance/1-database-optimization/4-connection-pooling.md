---
source_course: "django-performance"
source_lesson: "django-performance-database-connection-pooling"
---

# Database Connection Pooling

## Introduction

Connection pooling reuses database connections instead of creating new ones for each request, significantly improving performance.

## Key Concepts

- **Connection Pool**: A cache of reusable database connections that eliminates the overhead of establishing a new connection per request.
- **CONN_MAX_AGE**: Django's built-in setting that controls how long a database connection persists between requests (0 = close after each request, None = unlimited).
- **PgBouncer**: An external lightweight connection pooler for PostgreSQL that sits between Django and the database, managing connections at the infrastructure level.
- **Pool Mode**: PgBouncer's strategy for sharing connections — `transaction` mode (recommended for Django) releases connections back to the pool after each transaction.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

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



### Django 5.1+ Built-in Connection Pooling (PostgreSQL)

Django 5.1 introduced native connection pooling for PostgreSQL using psycopg3's pool. This is the recommended approach for Django 6:

```python
# settings.py - Django 6 with psycopg3
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'OPTIONS': {
            'pool': {
                'min_size': 2,
                'max_size': 4,
                'timeout': 10,
            }
        },
    }
}

# Simple boolean form:
# 'OPTIONS': {'pool': True}
```

This requires `psycopg[pool]` or `psycopg-pool` to be installed. It is ignored when using psycopg2.

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

## ASGI Caveat

When using ASGI servers (Uvicorn, Daphne), Django documentation recommends keeping `CONN_MAX_AGE` at `0` (the default) and using an external connection pooler like PgBouncer instead. Persistent connections under ASGI can lead to connection exhaustion because ASGI handles concurrency differently from WSGI.

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use CONN_MAX_AGE > 0 in production**: Reduces connection overhead.
2. **Match pool size to workers**: Gunicorn workers × connections per worker.
3. **Use transaction pooling for Django**: Compatible with ORM.

## Summary

Connection pooling improves performance by reusing database connections. Start with CONN_MAX_AGE, consider PgBouncer for larger deployments.

## Code Examples

**Configuring CONN_MAX_AGE for database connection persistence**

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


## Resources

- [Database Connections](https://docs.djangoproject.com/en/6.0/ref/databases/#connection-management) — Django connection management

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*