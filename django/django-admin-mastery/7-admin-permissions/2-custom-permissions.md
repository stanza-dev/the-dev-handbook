---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-custom-permissions"
---

# Custom Permissions

## Introduction

Define custom permissions beyond the default add/change/delete/view set for fine-grained access control.

## Key Concepts

**Model Meta permissions**: Define custom permissions on models.

**Permission codenames**: The identifier used to check permissions.

**has_<perm>_permission**: Custom permission methods on ModelAdmin.

## Real World Context

An e-commerce platform defines custom permissions like 'can_refund_order', 'can_feature_product', and 'can_view_financial_reports'. The customer service team gets refund permission, the marketing team gets feature permission, and only managers get financial report access. These granular permissions map cleanly to admin actions and custom views.

## Deep Dive

### Defining Custom Permissions

```python
# models.py
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)

    class Meta:
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature articles on homepage'),
            ('view_statistics', 'Can view article statistics'),
        ]
```

### Checking Custom Permissions

```python
from django.contrib.auth import get_permission_codename

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['publish']

    @admin.action(
        description='Publish selected',
        permissions=['publish']
    )
    def publish(self, request, queryset):
        queryset.update(status='published')

    def has_publish_permission(self, request):
        opts = self.opts
        codename = get_permission_codename('publish', opts)
        return request.user.has_perm(
            f'{opts.app_label}.{codename}'
        )
```

### Dynamic Permission Checks

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_actions(self, request):
        actions = super().get_actions(request)
        if not request.user.has_perm('blog.publish_article'):
            if 'publish' in actions:
                del actions['publish']
        return actions

    def get_list_display(self, request):
        fields = ['title', 'status', 'author']
        if request.user.has_perm('blog.view_statistics'):
            fields.append('view_count')
        return fields
```

## Common Pitfalls

1. **Forgetting to run migrations after adding permissions**: Custom permissions defined in Model Meta are created by migrations. Without running migrate, the permissions do not exist in the database and has_perm() always returns False.

2. **Using hardcoded permission strings**: Writing `request.user.has_perm('blog.publish_article')` with a typo silently returns False. Use get_permission_codename() or constants to avoid silent failures.

## Best Practices

1. **Use semantic permission names**: 'publish_article' not 'special_article'.
2. **Run migrations** after adding custom permissions.

## Summary

Custom permissions are defined in Model Meta. Use has_<perm>_permission on ModelAdmin for action-level control. Always run migrations after adding permissions.

## Resources

- [Custom Permissions](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/#custom-permissions) — Defining custom permissions

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*