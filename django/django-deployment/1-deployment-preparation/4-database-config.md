---
source_course: "django-deployment"
source_lesson: "django-deployment-database-config"
---

# Database Configuration

## Introduction

Production databases need connection pooling, SSL, proper credentials management, and backup strategies.

## Key Concepts

**Connection Pooling**: Reusing database connections for efficiency.

**CONN_MAX_AGE**: Django setting for persistent connections.

**SSL/TLS**: Encrypted database connections.

## Deep Dive

### PostgreSQL Configuration

```python
# settings/production.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ['DB_NAME'],
        'USER': os.environ['DB_USER'],
        'PASSWORD': os.environ['DB_PASSWORD'],
        'HOST': os.environ['DB_HOST'],
        'PORT': os.environ.get('DB_PORT', '5432'),
        'CONN_MAX_AGE': 60,  # Keep connections open
        'OPTIONS': {
            'sslmode': 'require',
        },
    }
}
```

### Using dj-database-url

```python
# pip install dj-database-url
import dj_database_url

DATABASES = {
    'default': dj_database_url.config(
        default=os.environ['DATABASE_URL'],
        conn_max_age=60,
        ssl_require=True
    )
}
```

### Connection Pooling with PgBouncer

```python
# When using PgBouncer
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ['DB_NAME'],
        'HOST': 'pgbouncer-host',
        'PORT': '6432',  # PgBouncer port
        'CONN_MAX_AGE': 0,  # Let PgBouncer handle pooling
        'DISABLE_SERVER_SIDE_CURSORS': True,
    }
}
```

### Read Replicas

```python
DATABASES = {
    'default': {...},
    'replica': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ['DB_NAME'],
        'HOST': os.environ['DB_REPLICA_HOST'],
        ...
    }
}

# Router for read/write splitting
DATABASE_ROUTERS = ['myproject.routers.PrimaryReplicaRouter']
```

## Best Practices

1. **Use CONN_MAX_AGE**: Reduces connection overhead.
2. **Enable SSL**: Always encrypt database connections.
3. **Use connection pooling**: PgBouncer for high-traffic sites.
4. **Separate credentials**: Different passwords for prod/staging.

## Summary

Configure databases with connection pooling (CONN_MAX_AGE), SSL encryption, and consider PgBouncer for high-traffic applications. Use environment variables for credentials.

## Resources

- [Database Settings](https://docs.djangoproject.com/en/6.0/ref/databases/) — Django database documentation

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*