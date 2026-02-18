---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-list-configuration"
---

# List View Configuration

## Introduction

The list view is the main interface for browsing model instances. Configure it for efficient data management.

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

## Summary

list_display controls columns. list_filter adds sidebar filters. search_fields enables search. date_hierarchy adds date navigation.

## Resources

- [List Display](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display) — List display options

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*