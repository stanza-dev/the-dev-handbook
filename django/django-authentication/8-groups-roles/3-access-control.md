---
source_course: "django-authentication"
source_lesson: "django-authentication-middleware-access-control"
---

# Middleware for Access Control

## Introduction

Use middleware for site-wide access control that applies to all views.

## Key Concepts

**Middleware**: Request/response processing layer.

**URL-Based Access**: Control access by URL pattern.

## Real World Context

A B2B platform has 50+ views that should only be accessible to paying subscribers. Instead of adding `@subscription_required` to every view, a single middleware checks the subscription status for any URL under `/app/`, keeping access control centralized and impossible to forget.

## Deep Dive

### Role-Required Middleware

```python
from django.http import HttpResponseForbidden

class RoleRequiredMiddleware:
    PROTECTED_PATHS = {
        '/admin/': ['admin'],
        '/dashboard/': ['admin', 'editor'],
        '/reports/': ['admin', 'analyst'],
    }
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        for path, roles in self.PROTECTED_PATHS.items():
            if request.path.startswith(path):
                if not request.user.is_authenticated:
                    return redirect('login')
                user_roles = request.user.groups.values_list('name', flat=True)
                if not any(role in user_roles for role in roles):
                    return HttpResponseForbidden('Access denied')
        return self.get_response(request)
```

### Subscription Middleware

```python
class SubscriptionMiddleware:
    PREMIUM_PATHS = ['/premium/', '/api/premium/']
    
    def __call__(self, request):
        if any(request.path.startswith(p) for p in self.PREMIUM_PATHS):
            if not getattr(request.user, 'has_premium', False):
                return redirect('upgrade')
        return self.get_response(request)
```

## Common Pitfalls

1. **Blocking static files and login pages**: If your middleware applies to all URLs, unauthenticated users cannot reach the login page or load CSS/JS. Always exclude authentication URLs and static file paths.
2. **Not ordering middleware correctly**: Access control middleware must come after `AuthenticationMiddleware` so that `request.user` is available. Placing it before causes `AttributeError`.
3. **Heavy database queries in middleware**: Middleware runs on every request. Fetching group memberships or subscription status from the database on each request adds latency. Cache the result in the session or use Django's caching framework.

## Best Practices

1. **Keep middleware focused**: Single responsibility.
2. **Use for cross-cutting concerns**: Site-wide rules.

## Summary

Middleware enforces site-wide access rules. Use for role-based URL protection. Keep logic simple and focused.

## Resources

- [Middleware](https://docs.djangoproject.com/en/6.0/topics/http/middleware/) — Django middleware

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*