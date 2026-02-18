---
source_course: "django-authentication"
source_lesson: "django-authentication-request-user"
---

# Understanding request.user

## Introduction

The request.user attribute is your gateway to the current user. Understanding how it works is essential for building authenticated features.

## Key Concepts

**request.user**: The current user (User or AnonymousUser).

**is_authenticated**: Boolean property for auth status.

## Deep Dive

### Checking Authentication

```python
def my_view(request):
    if request.user.is_authenticated:
        # User is logged in
        username = request.user.username
        email = request.user.email
    else:
        # AnonymousUser
        pass
```

### AnonymousUser Properties

```python
from django.contrib.auth.models import AnonymousUser

anon = AnonymousUser()
anon.is_authenticated  # False
anon.is_anonymous      # True
anon.id                # None
anon.pk                # None
anon.username          # '' (empty string)
anon.has_perm('any')   # False
```

### Common Patterns

```python
# Get user or None for optional relations
author = request.user if request.user.is_authenticated else None

# Filter by user
articles = Article.objects.filter(
    author=request.user
) if request.user.is_authenticated else Article.objects.none()
```

## Best Practices

1. **Always check is_authenticated**: Before accessing user properties.
2. **Don't compare to None**: Use is_authenticated instead.
3. **Handle AnonymousUser gracefully**: Don't assume logged in.

## Summary

request.user is always available - it's either a User or AnonymousUser. Always check is_authenticated before accessing user-specific data.

## Resources

- [Request Object](https://docs.djangoproject.com/en/6.0/ref/request-response/#django.http.HttpRequest.user) — Django request.user documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*