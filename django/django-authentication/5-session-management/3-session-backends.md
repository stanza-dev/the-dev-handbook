---
source_course: "django-authentication"
source_lesson: "django-authentication-session-backends"
---

# Session Backends

## Introduction

Choose the right session backend based on your performance and reliability needs.

## Key Concepts

**Database Backend**: Default, stores in DB.

**Cache Backend**: Fast, stores in Redis/Memcached.

## Deep Dive

### Backend Comparison

```python
# Database (default) - reliable but slower
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# Cache - fastest, may lose sessions on restart
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

# Cached DB - fast reads, reliable writes
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'

# Signed cookies - no server storage, limited size
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
```

### Redis Sessions

```python
# pip install django-redis

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}

SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

## Best Practices

1. **Use cached_db for production**: Balance of speed and reliability.
2. **Use Redis with persistence**: Prevent session loss.

## Summary

Cache backend is fastest. Cached_db balances speed and reliability. Redis is the best production choice.

## Resources

- [Session Backends](https://docs.djangoproject.com/en/6.0/topics/http/sessions/#configuring-the-session-engine) — Session backends

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*