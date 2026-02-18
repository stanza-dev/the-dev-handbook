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

## Best Practices

1. **Keep middleware focused**: Single responsibility.
2. **Use for cross-cutting concerns**: Site-wide rules.

## Summary

Middleware enforces site-wide access rules. Use for role-based URL protection. Keep logic simple and focused.

## Resources

- [Middleware](https://docs.djangoproject.com/en/6.0/topics/http/middleware/) — Django middleware

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*