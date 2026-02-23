---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-custom-admin-site"
---

# Custom Admin Site

## Introduction

Django's default admin site works out of the box, but you can customize its header, title, and behavior or create entirely separate admin sites for different audiences. This lesson covers site-level customization from simple branding to multiple admin interfaces.

## Key Concepts

**AdminSite**: The class representing an admin site instance.

**site_header**: Text displayed in the admin page header.

**site_title**: Text used in the browser tab title.

**index_title**: Heading on the admin index page.

**each_context()**: Method to inject variables into every admin template.

## Real World Context

A multi-tenant SaaS application serves both content editors and system administrators. Content editors need a simplified admin showing only articles and pages, while system admins need access to users, permissions, and audit logs. Creating two AdminSite subclasses with different registered models and has_permission() checks gives each audience a focused interface without any custom views.

## Deep Dive

You can customize the default admin site or create multiple admin sites for different purposes.

### Basic Site Customization

```python
# admin.py
from django.contrib import admin

# Customize the default site
admin.site.site_header = 'My Company Admin'
admin.site.site_title = 'My Company Admin Portal'
admin.site.index_title = 'Welcome to the Admin Panel'
```

### Custom AdminSite Class

```python
# admin.py
from django.contrib.admin import AdminSite
from django.utils.translation import gettext_lazy as _


class MyAdminSite(AdminSite):
    site_header = _('My Company Administration')
    site_title = _('My Company Admin')
    index_title = _('Dashboard')
    
    def each_context(self, request):
        """Add extra context to every admin page."""
        context = super().each_context(request)
        context['custom_var'] = 'Custom value'
        return context
    
    def has_permission(self, request):
        """Control who can access admin."""
        return request.user.is_active and request.user.is_staff


my_admin_site = MyAdminSite(name='myadmin')
```

### Register Models with Custom Site

```python
# admin.py
from .models import Article, Author

@my_admin_site.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'status']


my_admin_site.register(Author)
```

```python
# urls.py
from django.urls import path
from .admin import my_admin_site

urlpatterns = [
    path('myadmin/', my_admin_site.urls),
]
```

### Multiple Admin Sites

```python
# admin.py
class PublicAdminSite(AdminSite):
    """Admin site for content editors."""
    site_header = 'Content Management'
    
    def has_permission(self, request):
        return request.user.groups.filter(name='Editors').exists()


class SuperAdminSite(AdminSite):
    """Admin site for superusers only."""
    site_header = 'System Administration'
    
    def has_permission(self, request):
        return request.user.is_superuser


public_admin = PublicAdminSite(name='public_admin')
super_admin = SuperAdminSite(name='super_admin')

# Register different models to different sites
public_admin.register(Article, ArticleAdmin)
public_admin.register(Page, PageAdmin)

super_admin.register(User, UserAdmin)
super_admin.register(Group)
super_admin.register(SystemSettings)
```

### Custom Admin Templates

Override templates by placing them in your templates directory:

```
templates/
└── admin/
    ├── base_site.html          # Override base template
    ├── index.html              # Override index page
    └── myapp/
        └── article/
            ├── change_list.html  # Override for specific model
            └── change_form.html
```

```html
<!-- templates/admin/base_site.html -->
{% extends "admin/base.html" %}
{% load static %}

{% block branding %}
<h1 id="site-name">
    <a href="{% url 'admin:index' %}">
        <img src="{% static 'img/logo.png' %}" alt="Logo" height="40">
        {{ site_header|default:_("Django administration") }}
    </a>
</h1>
{% endblock %}

{% block extrastyle %}
{{ block.super }}
<link rel="stylesheet" href="{% static 'css/admin-custom.css' %}">
{% endblock %}
```

### Custom CSS

```css
/* static/css/admin-custom.css */
:root {
    --primary: #092e20;
    --secondary: #44b78b;
    --accent: #0c4b33;
}

#header {
    background: var(--primary);
}

.module h2, .module caption {
    background: var(--secondary);
}

a:link, a:visited {
    color: var(--accent);
}
```

## Common Pitfalls

1. **Forgetting to set a unique name parameter**: Each AdminSite must have a unique name for URL namespacing. Without it, URL reversing breaks when you have multiple admin sites.

2. **Not updating urls.py**: Creating a custom AdminSite without adding its urls to urlpatterns means it is unreachable. Each site needs its own path entry.

3. **Overriding templates globally instead of per-site**: Template overrides in `templates/admin/` affect all admin sites. Use per-app template directories or set change_list_template per ModelAdmin to target specific sites.

## Best Practices

1. **Use each_context() for global variables**: Instead of repeating context in every custom view, override each_context() to add company name, version, or feature flags site-wide.

2. **Subclass AdminSite instead of patching admin.site**: Setting admin.site.site_header works for simple cases, but subclassing gives you full control over has_permission(), get_urls(), and template context.

3. **Keep template overrides minimal**: Extend the default admin templates and override only the blocks you need. Replacing entire templates breaks on Django upgrades.

## Summary

- Customize the default admin site with admin.site.site_header and similar attributes.
- Subclass AdminSite for full control over behavior and permissions.
- Create multiple admin sites to serve different user audiences.
- Override templates by placing files in the correct template directory hierarchy.
- Use each_context() to inject shared variables across all admin pages.

## Code Examples

**Custom AdminSite with customized header, title, and index title**

```python
from django.contrib.admin import AdminSite

class MyAdminSite(AdminSite):
    site_header = 'My Company Admin'
    site_title = 'My Company Admin Portal'
    index_title = 'Welcome to the Admin Portal'

admin_site = MyAdminSite(name='myadmin')
```


## Resources

- [AdminSite](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#adminsite-objects) — AdminSite customization reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*