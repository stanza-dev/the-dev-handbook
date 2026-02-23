---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-display-decorator"
---

# Custom Display Methods

## Introduction

Use the @admin.display decorator to create custom columns in the admin change list.

## Key Concepts

**@admin.display**: Decorator for display methods.

**boolean**: Show checkmarks instead of True/False.

**ordering**: Enable column sorting on computed fields.

## Real World Context

A product catalog admin needs to show a colored stock indicator (green for in-stock, red for out-of-stock) and a thumbnail image directly in the change list. Using @admin.display with format_html(), the admin becomes a visual product browser where team members can scan inventory status without clicking into each product.

## Deep Dive

### Basic Display Methods

```python
from django.contrib import admin
from django.utils.html import format_html

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = [
        'title', 'author_name', 'colored_status',
        'is_recent', 'word_count'
    ]

    @admin.display(description='Author', ordering='author__username')
    def author_name(self, obj):
        return obj.author.get_full_name() or obj.author.username

    @admin.display(description='Status')
    def colored_status(self, obj):
        colors = {
            'published': 'green',
            'draft': 'orange',
            'archived': 'gray',
        }
        color = colors.get(obj.status, 'black')
        return format_html(
            '<span style="color: {};">{}</span>',
            color, obj.status.title()
        )

    @admin.display(boolean=True, description='Recent?')
    def is_recent(self, obj):
        from django.utils import timezone
        from datetime import timedelta
        return obj.pub_date >= timezone.now() - timedelta(days=7)

    @admin.display(description='Words', ordering='body')
    def word_count(self, obj):
        return len(obj.body.split())
```

### Display with Links

```python
from django.urls import reverse
from django.utils.html import format_html

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['text_preview', 'article_link', 'author', 'created_at']

    @admin.display(description='Comment')
    def text_preview(self, obj):
        return obj.text[:50] + '...' if len(obj.text) > 50 else obj.text

    @admin.display(description='Article')
    def article_link(self, obj):
        url = reverse('admin:blog_article_change', args=[obj.article.pk])
        return format_html('<a href="{}">{}</a>', url, obj.article.title)
```

## Common Pitfalls

1. **Using f-strings or string concatenation for HTML output**: This creates XSS vulnerabilities. Always use format_html() or mark_safe() with properly escaped values.

2. **Forgetting to add the ordering parameter**: Without ordering, computed columns cannot be sorted by clicking the column header. Set ordering to the underlying database field when possible.

## Best Practices

1. **Use format_html()**: Never use f-strings for HTML to prevent XSS.
2. **Add ordering**: Make computed columns sortable where possible.

## Summary

@admin.display decorates methods for use in list_display. Use boolean=True for checkmarks, ordering for sortability, and format_html for safe HTML output.

## Resources

- [admin.display](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#the-display-decorator) — @admin.display decorator reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*