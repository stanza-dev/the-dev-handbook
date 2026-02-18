---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-fieldsets"
---

# Fieldsets and Form Layout

## Introduction

Organize the change form into logical sections using fieldsets.

## Deep Dive

### Basic Fieldsets

```python
fieldsets = [
    (None, {
        'fields': ['title', 'slug', 'body']
    }),
    ('Publishing', {
        'fields': ['author', 'status', 'pub_date'],
        'classes': ['collapse'],  # Collapsible
    }),
    ('SEO', {
        'fields': ['meta_description', 'meta_keywords'],
        'classes': ['wide'],
    }),
]
```

### Special Options

```python
fieldsets = [
    ('Advanced', {
        'classes': ['collapse', 'wide'],
        'description': 'Optional advanced settings',
        'fields': ['custom_css', 'custom_js'],
    }),
]

readonly_fields = ['created_at', 'updated_at']
prepopulated_fields = {'slug': ('title',)}
```

## Summary

Fieldsets group related fields. Use 'collapse' for optional sections. Use 'wide' for fields needing more space.

## Resources

- [Fieldsets](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets) — Fieldset configuration

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*