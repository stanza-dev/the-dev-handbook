---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-list-configuration"
---

# List View Configuration

## Introduction

The list view is the main interface for browsing model instances. Configure it for efficient data management.

## Key Concepts

**list_display**: Columns shown in the change list view.

**list_display_links**: Fields that link to the change form.

**list_editable**: Fields editable directly in the list view.

**@admin.display**: Decorator for custom display methods with sorting and formatting.

## Real World Context

A logistics company tracks thousands of shipments. Dispatchers need to see status, origin, destination, and delivery date at a glance without opening each record. list_display shows all key fields as columns, list_filter adds a status sidebar, and list_editable lets them update status directly in the table, turning the admin into a lightweight operations dashboard.

## Deep Dive

### List Display

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'status', 'pub_date', 'views']
    list_display_links = ['title']  # Clickable fields
    list_editable = ['status']       # Edit in list view
```

### Filtering and Search

```python
list_filter = ['status', 'pub_date', ('author', admin.RelatedOnlyFieldListFilter)]
search_fields = ['title', 'body', 'author__username']
date_hierarchy = 'pub_date'  # Date navigation bar
```

### Custom Display Methods

```python
@admin.display(description='Status', ordering='status')
def status_badge(self, obj):
    colors = {'draft': 'gray', 'published': 'green'}
    return format_html('<span style="color: {};">{}</span>',
                       colors.get(obj.status, 'black'), obj.status)
```

## Common Pitfalls

1. **Including the first list_display field in list_editable**: The first field (or the field in list_display_links) must remain a link to the change page. Django will raise an ImproperlyConfigured error if you try to make it editable.

2. **Adding too many columns to list_display**: More than 7-8 columns makes the table unreadable and can slow page load if custom methods perform database queries per row.

## Best Practices

1. **Use @admin.display(ordering=...) on computed columns**: This allows the column to be sortable by clicking its header, backed by an actual database field.

2. **Use select_related or list_select_related for ForeignKey columns**: Prevents N+1 queries when displaying related model fields in each row.

## Summary

list_display controls columns. list_filter adds sidebar filters. search_fields enables search. date_hierarchy adds date navigation.

## Resources

- [List Display](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display) — List display options

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*