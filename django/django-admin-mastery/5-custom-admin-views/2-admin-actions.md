---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-bulk-admin-actions"
---

# Custom Admin Actions

## Introduction

Admin actions let users perform bulk operations on selected objects from the change list.

## Key Concepts

**Action function**: A function that receives the ModelAdmin, request, and queryset.

**@admin.action**: Decorator for describing actions.

**permissions**: Control which users can perform actions.

## Real World Context

In a content moderation system, moderators review hundreds of user-submitted posts daily. A 'Approve selected' action lets them select multiple posts and approve them in one click. A 'Reject and notify' action sends automated rejection emails to submitters. These bulk actions turn the admin change list into an efficient moderation queue that replaces what would otherwise require a custom internal tool.

## Deep Dive

### Basic Action

```python
from django.contrib import admin
from .models import Article

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['make_published', 'make_draft']

    @admin.action(description='Mark selected as published')
    def make_published(self, request, queryset):
        updated = queryset.update(status='published')
        self.message_user(
            request,
            f'{updated} articles marked as published.'
        )

    @admin.action(description='Mark selected as draft')
    def make_draft(self, request, queryset):
        queryset.update(status='draft')
```

### Action with Permission Check

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['publish_selected']

    @admin.action(
        description='Publish selected articles',
        permissions=['publish']
    )
    def publish_selected(self, request, queryset):
        queryset.update(status='published')

    def has_publish_permission(self, request):
        return request.user.has_perm('blog.publish_article')
```

### Action with Confirmation

```python
from django.contrib import admin, messages

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['delete_unpublished']

    @admin.action(description='Delete all unpublished articles')
    def delete_unpublished(self, request, queryset):
        unpublished = queryset.filter(status='draft')
        count = unpublished.count()
        if count == 0:
            self.message_user(request, 'No drafts selected.', messages.WARNING)
            return
        unpublished.delete()
        self.message_user(request, f'Deleted {count} draft articles.')
```

## Best Practices

1. **Use @admin.action** decorator for descriptions and permissions.
2. **Provide feedback**: Always call message_user() after an action.

## Common Pitfalls

1. **Not testing actions with large querysets**: An action that iterates over queryset items individually (e.g., sending emails in a loop) can time out on large selections. Use queryset.update() for database changes and batch processing for external operations.

2. **Forgetting the message_user() feedback**: Actions that complete without feedback leave admins uncertain. Always call self.message_user() with a clear count of affected objects.

## Summary

Admin actions process selected objects in bulk. Use the @admin.action decorator. Add permission checks with the permissions parameter.

## Resources

- [Admin Actions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/actions/) — Django admin actions

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*