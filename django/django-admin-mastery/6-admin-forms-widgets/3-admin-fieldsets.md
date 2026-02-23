---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-fieldsets"
---

# Admin Fieldsets

## Introduction

Fieldsets organize form fields into logical groups with optional collapsibility in the admin change form.

## Key Concepts

**fieldsets**: Group and organize form fields.

**classes**: CSS classes like 'collapse' and 'wide'.

**description**: Help text for each fieldset.

## Real World Context

A CRM application has a Contact model with personal details, company information, communication preferences, and internal notes. Without fieldsets, all 20+ fields appear in a single long form. With fieldsets grouping them into 'Personal Info', 'Company', 'Preferences' (collapsed), and 'Internal' (collapsed, superusers only via get_fieldsets), each user sees an organized form tailored to their needs.

## Deep Dive

### Basic Fieldsets

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {
            'fields': ('title', 'slug', 'author'),
        }),
        ('Content', {
            'fields': ('body', 'summary', 'featured_image'),
            'description': 'The main content of the article.',
        }),
        ('Publishing', {
            'fields': ('status', 'pub_date', 'category', 'tags'),
            'classes': ('collapse',),
            'description': 'Publishing options and categorization.',
        }),
        ('SEO', {
            'fields': ('meta_title', 'meta_description', 'canonical_url'),
            'classes': ('collapse',),
        }),
    ]
```

### Dynamic Fieldsets

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_fieldsets(self, request, obj=None):
        fieldsets = [
            (None, {'fields': ('title', 'body')}),
        ]

        if obj:  # Editing existing object
            fieldsets.append(('Publishing', {
                'fields': ('status', 'pub_date'),
            }))

        if request.user.is_superuser:
            fieldsets.append(('Advanced', {
                'fields': ('author', 'featured'),
                'classes': ('collapse',),
            }))

        return fieldsets
```

### Horizontal Filter for M2M

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    filter_horizontal = ('tags',)  # Side-by-side widget
    # OR
    filter_vertical = ('categories',)  # Stacked widget
```

## Common Pitfalls

1. **Using both fields and fieldsets on the same ModelAdmin**: Django raises an error if both are set. Choose one: fields for a flat list, fieldsets for grouped sections.

2. **Forgetting to include all model fields**: Fields not listed in any fieldset will not appear on the form and will use their default values. This can silently drop user input for fields that existed before you added fieldsets.

## Best Practices

1. **Group related fields**: Keep forms organized.
2. **Collapse rarely-used sections**: Reduce visual noise.
3. **Use descriptions**: Guide admins through complex forms.

## Summary

Fieldsets organize admin forms into sections. Use 'collapse' to hide optional sections. Override get_fieldsets() for dynamic layouts based on user or object state.

## Resources

- [ModelAdmin.fieldsets](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets) — Admin fieldsets reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*