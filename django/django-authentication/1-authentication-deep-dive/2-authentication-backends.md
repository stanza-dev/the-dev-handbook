---
source_course: "django-authentication"
source_lesson: "django-authentication-authentication-backends"
---

# Authentication Backends

## Introduction

Authentication backends are the pluggable system Django uses to verify user credentials. You can customize how users authenticate by writing custom backends.

## Key Concepts

**Backend**: A class with `authenticate()` and `get_user()` methods.

**ModelBackend**: Django's default backend that checks username/password against the database.

**Backend Order**: Django tries each backend in `AUTHENTICATION_BACKENDS` until one succeeds.

## Real World Context

Custom backends enable email-based login, LDAP/Active Directory integration, token authentication, and multi-tenant setups. Most production Django apps customize their authentication backend.

## Deep Dive

### Default ModelBackend

```python
# Django's built-in backend
from django.contrib.auth.backends import ModelBackend

class ModelBackend:
    def authenticate(self, request, username=None, password=None, **kwargs):
        UserModel = get_user_model()
        try:
            user = UserModel._default_manager.get_by_natural_key(username)
        except UserModel.DoesNotExist:
            UserModel().set_password(password)  # Timing attack protection
            return None
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
        return None
```

### Email Authentication Backend

```python
from django.contrib.auth import get_user_model
from django.contrib.auth.backends import ModelBackend

class EmailBackend(ModelBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        UserModel = get_user_model()
        try:
            # Allow login with email
            user = UserModel.objects.get(email=username)
        except UserModel.DoesNotExist:
            return None

        if user.check_password(password) and self.user_can_authenticate(user):
            return user
        return None
```

### Configuration

```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'myapp.backends.EmailBackend',
    'django.contrib.auth.backends.ModelBackend',  # Fallback
]

# Now authenticate() checks EmailBackend first, then ModelBackend
```

### Backend with Rate Limiting

```python
from django.core.cache import cache

class RateLimitedBackend(ModelBackend):
    MAX_ATTEMPTS = 5
    LOCKOUT_DURATION = 300  # 5 minutes

    def authenticate(self, request, username=None, password=None, **kwargs):
        cache_key = f'login_attempts_{username}'
        attempts = cache.get(cache_key, 0)

        if attempts >= self.MAX_ATTEMPTS:
            return None  # Locked out

        user = super().authenticate(request, username=username, password=password, **kwargs)

        if user is None:
            cache.set(cache_key, attempts + 1, self.LOCKOUT_DURATION)
        else:
            cache.delete(cache_key)

        return user
```

## Common Pitfalls

1. **Forgetting `get_user()`**: Custom backends must implement both `authenticate()` and optionally `get_user()`.
2. **Not handling timing attacks**: Always run `set_password()` even when user not found.
3. **Wrong backend order**: More specific backends should come first.

## Best Practices

1. **Extend ModelBackend**: Instead of writing from scratch, extend the default.
2. **Keep the default as fallback**: Add `ModelBackend` as the last backend.
3. **Use `user_can_authenticate()`**: Respects the `is_active` flag.

## Summary

Authentication backends let you customize credential verification. Extend `ModelBackend` for common cases. Configure `AUTHENTICATION_BACKENDS` in settings. Backends are checked in order until one returns a user.

## Resources

- [Customizing Authentication](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/) — Official guide to customizing authentication

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*