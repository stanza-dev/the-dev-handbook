---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-search-configuration"
---

# Search Configuration

## Introduction

Configure the admin search to help users find records quickly across related fields.

## Key Concepts

**search_fields**: Fields to search against.

**search_help_text**: Hint text shown above the search box.

**get_search_results()**: Override for custom search logic.

## Real World Context

A customer support portal lets agents search for users by name, email, or ticket ID. Without search_fields, agents must manually browse paginated lists. With properly configured search_fields using the `^` prefix on email (startswith) and `=` on ticket ID (exact match), the search becomes both fast and precise even with millions of records.

## Deep Dive

### Basic Search Fields

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    search_fields = [
        'title',
        'body',
        'author__username',
        'author__email',
        'tags__name',
    ]
    search_help_text = 'Search by title, body, author, or tag'
```

### Search Field Prefixes

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    search_fields = [
        '^title',       # startswith
        '=author__username',  # exact match
        'body',         # icontains (default)
        '@body',        # full-text search (PostgreSQL)
    ]
```

### Custom Search Logic

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_search_results(self, request, queryset, search_term):
        queryset, may_have_duplicates = super().get_search_results(
            request, queryset, search_term
        )

        # Also search by ID if numeric
        if search_term.isdigit():
            queryset |= self.model.objects.filter(pk=search_term)

        return queryset, may_have_duplicates
```

## Common Pitfalls

1. **Using icontains (default) on large text fields without indexes**: The default search uses icontains which cannot use database indexes. For large datasets, use `^` (startswith) or `@` (full-text search on PostgreSQL) prefixes.

2. **Not setting search_help_text**: Users do not know which fields are searchable unless you tell them. Always set search_help_text to describe what can be searched.

## Best Practices

1. **Add search_help_text**: Users need to know what they can search.
2. **Use prefixes** for large datasets to optimize queries.

## Summary

search_fields enables the search box. Use prefixes (^, =, @) to control matching. Override get_search_results() for custom logic.

## Resources

- [ModelAdmin.search_fields](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields) — Admin search configuration

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*