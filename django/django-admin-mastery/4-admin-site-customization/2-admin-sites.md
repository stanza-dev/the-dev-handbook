---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-multiple-admin-sites"
---

# Multiple Admin Sites

## Introduction

Django supports multiple admin sites, each with different configurations and registered models.

## Key Concepts

**AdminSite**: The admin site class that can be subclassed.

**name parameter**: Unique identifier for URL namespacing.

**Separate registrations**: Each site has its own registered models.

## Real World Context

A university application has three user groups: admissions staff, faculty, and IT administrators. Each group needs different models and permissions. Instead of one cluttered admin with complex permission logic, three AdminSite instances (admissions_admin, faculty_admin, it_admin) each register only the relevant models and enforce group-based access via has_permission().

## Deep Dive

### Creating Multiple Sites

```python
# admin.py
from django.contrib.admin import AdminSite

class PublicAdminSite(AdminSite):
    site_header = 'Public Admin'
    site_title = 'Public Admin'
    index_title = 'Content Management'

class InternalAdminSite(AdminSite):
    site_header = 'Internal Admin'
    site_title = 'Internal Tools'
    index_title = 'Internal Dashboard'

public_admin = PublicAdminSite(name='public_admin')
internal_admin = InternalAdminSite(name='internal_admin')
```

### Registering Models per Site

```python
# Register different models on different sites
from .models import Article, Page, User, AuditLog

public_admin.register(Article, ArticleAdmin)
public_admin.register(Page, PageAdmin)

internal_admin.register(User, UserAdmin)
internal_admin.register(AuditLog, AuditLogAdmin)
```

### URL Configuration

```python
# urls.py
from .admin import public_admin, internal_admin

urlpatterns = [
    path('admin/', admin.site.urls),
    path('content-admin/', public_admin.urls),
    path('internal/', internal_admin.urls),
]
```

## Common Pitfalls

1. **Registering a model on multiple sites without separate ModelAdmin classes**: Using the same ModelAdmin instance on two sites can cause unexpected shared state. Always create distinct ModelAdmin classes or use the @site.register decorator per site.

2. **Forgetting URL namespacing**: Without unique name parameters, reverse('admin:index') becomes ambiguous. Always pass a unique name when instantiating AdminSite.

## Best Practices

1. **Use unique names**: Each AdminSite needs a unique name for URL resolution.
2. **Separate by audience**: Content editors vs. system administrators.

## Summary

Multiple admin sites allow different interfaces for different user types. Each site has independent model registrations and URL patterns.

## Resources

- [Multiple Admin Sites](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#multiple-admin-sites-in-the-same-urlconf) — Running multiple admin sites

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*