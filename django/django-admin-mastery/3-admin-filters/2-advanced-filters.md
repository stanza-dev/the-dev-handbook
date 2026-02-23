---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-advanced-filters"
---

# Advanced Filter Techniques

## Introduction

Beyond SimpleListFilter, Django admin provides several ways to create sophisticated filtering in your admin views.

## Key Concepts

**FieldListFilter**: Base class for field-based filters.

**RelatedFieldListFilter**: Filter by related model fields.

**EmptyFieldListFilter**: Filter by empty/non-empty values.

## Real World Context

An analytics dashboard needs to filter reports by multiple dimensions: empty description fields, specific author countries, and chained status-author combinations. Built-in field filters cannot express these cross-model or empty-value conditions. Using EmptyFieldListFilter, AllValuesFieldListFilter, and chained SimpleListFilter subclasses, you can build precise multi-dimensional filtering.

## Deep Dive

### Filter by Related Model

```python
from django.contrib import admin

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_filter = [
        'status',
        'pub_date',
        ('author', admin.RelatedOnlyFieldListFilter),
        ('category__name', admin.AllValuesFieldListFilter),
    ]
```

### Filter by Empty Fields

```python
from django.contrib.admin import EmptyFieldListFilter

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_filter = [
        ('description', EmptyFieldListFilter),
        ('pub_date', EmptyFieldListFilter),
    ]
```

### Chained Filters

```python
class StatusAuthorFilter(admin.SimpleListFilter):
    title = 'published by author'
    parameter_name = 'pub_author'

    def lookups(self, request, model_admin):
        authors = set()
        for article in model_admin.model.objects.filter(status='published'):
            authors.add((article.author.pk, str(article.author)))
        return sorted(authors, key=lambda x: x[1])

    def queryset(self, request, queryset):
        if self.value():
            return queryset.filter(
                status='published',
                author__pk=self.value()
            )
        return queryset
```

## Common Pitfalls

1. **Chained filters that produce N+1 queries**: A filter whose lookups() iterates over all objects (e.g., `for article in model_admin.model.objects.all()`) causes performance issues on large tables. Use values_list() with distinct() instead.

2. **Forgetting to handle the 'All' case**: When self.value() returns None, your queryset() must return the unfiltered queryset. Applying a filter when value is None removes all results.

## Best Practices

1. **Use RelatedOnlyFieldListFilter** for ForeignKey fields with many options.
2. **Keep filter lists short**: Too many filters overwhelm users.

## Summary

Use field-specific filter classes for common patterns. RelatedOnlyFieldListFilter shows only related values. EmptyFieldListFilter filters by null/blank values.

## Resources

- [List Filter Reference](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/filters/) — Django admin filter classes

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*