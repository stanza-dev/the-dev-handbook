---
source_course: "django-authentication"
source_lesson: "django-authentication-custom-permissions"
---

# Custom Permissions

## Introduction

Beyond CRUD, apps need custom permissions like 'publish' or 'archive'.

## Key Concepts

**Meta.permissions**: Define in model class.

## Real World Context

A CMS needs a `publish_article` permission separate from `change_article` so editors can draft content but only senior editors can make it live. Custom permissions let you model these real business workflows without conflating editing and publishing.

## Deep Dive

### Defining Custom Permissions

```python
class Article(models.Model):
    class Meta:
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature on homepage'),
        ]
```

### Using Custom Permissions

```python
@permission_required('blog.publish_article', raise_exception=True)
def publish(request, pk):
    article.status = 'published'
    article.save()
```

## Common Pitfalls

1. **Forgetting to run migrations after adding `Meta.permissions`**: Custom permissions are created by migrations. If you skip `makemigrations` and `migrate`, the permission does not exist in the database and `has_perm()` always returns `False`.
2. **Using wrong app label in permission strings**: The format is `app_label.codename` (e.g., `blog.publish_article`). Using the model name instead of the app label causes silent failures.
3. **Not clearing the permission cache**: After assigning permissions at runtime, the user object caches its permissions. You must refetch the user (`User.objects.get(pk=user.pk)`) or delete `user._perm_cache` to see the change.

## Best Practices

1. **Define permissions in `Meta.permissions`**: This ensures they are created by migrations and tracked in version control.
2. **Use `permission_required` with `raise_exception=True`**: This returns a 403 Forbidden instead of silently redirecting to the login page for authenticated users who lack the permission.
3. **Group custom permissions logically**: Name codenames with a consistent pattern like `verb_model` (e.g., `publish_article`, `archive_article`) so they are easy to discover in the admin.

## Summary

Define custom permissions in Meta. Use permission_required decorator.

## Resources

- [Custom Permissions](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/#custom-permissions) — Custom permissions

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*