---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-fieldsets"
---

# Fieldsets and Form Layout

## Introduction

Organize the change form into logical sections using fieldsets.

## Key Concepts

**fieldsets**: A list of two-tuples that group fields into named sections.

**classes**: CSS classes applied to the fieldset, such as 'collapse' and 'wide'.

**description**: Help text displayed at the top of a fieldset.

**readonly_fields**: Fields displayed but not editable on the change form.

## Real World Context

A SaaS product has a complex settings model with 30+ fields. Without fieldsets, the change form is a single overwhelming scroll. By grouping fields into 'General', 'Billing', 'Feature Flags', and 'Advanced' fieldsets with the advanced ones collapsed, the admin form becomes organized and manageable for different team members who only need certain sections.

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

## Common Pitfalls

1. **Listing a field in both fieldsets and fields**: Django does not allow both attributes on the same ModelAdmin. Use fieldsets when you need sections or use fields for a flat layout, but never both.

2. **Forgetting to include readonly_fields in a fieldset**: If a field is in readonly_fields but not listed in any fieldset, it will not appear on the form at all.

## Best Practices

1. **Collapse rarely-used sections**: Use `'classes': ['collapse']` for advanced or optional fields to reduce visual noise and keep the form focused.

2. **Group inline tuple fields for side-by-side layout**: Place related short fields like `('start_date', 'end_date')` in a tuple within the fields list to render them on the same row.

## Summary

Fieldsets group related fields. Use 'collapse' for optional sections. Use 'wide' for fields needing more space.

## Resources

- [Fieldsets](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets) — Fieldset configuration

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*