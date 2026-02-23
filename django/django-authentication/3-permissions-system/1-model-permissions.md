---
source_course: "django-authentication"
source_lesson: "django-authentication-model-permissions"
---

# Model Permissions

## Introduction

Django automatically creates four permissions (add, change, delete, view) for every model. Understanding how to check, assign, and customize these permissions is the foundation of access control in any Django application.

## Key Concepts

**Default Permissions**: Django creates `add_<model>`, `change_<model>`, `delete_<model>`, and `view_<model>` for each model.

**Permission String**: Format is `app_label.codename` (e.g., `blog.add_article`).

**`has_perm()`**: Method on User to check a single permission.

**`permission_required`**: Decorator that restricts view access to users with specific permissions.

**`Meta.permissions`**: Define custom permissions beyond the default four.

## Real World Context

In a multi-author blog, only staff members should be able to delete articles, while regular authors can only add and edit their own. Using Django's permission system with `@permission_required('blog.delete_article')` on the delete view enforces this rule without any custom logic in the view itself.

## Deep Dive

Django automatically creates permissions for each model. Learn to use and customize them.

## Default Model Permissions

For each model, Django creates:
- `add_<model>` - Can add instances
- `change_<model>` - Can edit instances
- `delete_<model>` - Can delete instances
- `view_<model>` - Can view instances

```python
# For model Article in app 'blog'
# Permissions created:
# - blog.add_article
# - blog.change_article
# - blog.delete_article
# - blog.view_article
```

## Checking Permissions

```python
# Check if user has permission
user.has_perm('blog.add_article')       # Single permission
user.has_perm('blog.change_article')

# Check multiple permissions (all required)
user.has_perms(['blog.add_article', 'blog.change_article'])

# Get all permissions
user.get_all_permissions()
# {'blog.add_article', 'blog.change_article', ...}

# Get group permissions
user.get_group_permissions()
```

## In Views

```python
from django.contrib.auth.decorators import permission_required

# Single permission
@permission_required('blog.add_article')
def create_article(request):
    pass

# Multiple permissions (all required)
@permission_required(['blog.add_article', 'blog.change_article'])
def manage_article(request):
    pass

# Raise 403 instead of redirecting to login
@permission_required('blog.delete_article', raise_exception=True)
def delete_article(request, pk):
    pass

# Manual check in view
def article_view(request):
    if not request.user.has_perm('blog.view_article'):
        return HttpResponseForbidden("Permission denied")
    # ...
```

## In Templates

```html
{% if perms.blog.add_article %}
    <a href="{% url 'article_create' %}">Create Article</a>
{% endif %}

{% if perms.blog.change_article %}
    <a href="{% url 'article_edit' article.pk %}">Edit</a>
{% endif %}

{% if perms.blog %}
    <!-- User has ANY permission in blog app -->
{% endif %}
```

## Custom Model Permissions

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    is_published = models.BooleanField(default=False)
    
    class Meta:
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature articles on homepage'),
            ('archive_article', 'Can archive articles'),
        ]

# After migration:
# - blog.publish_article
# - blog.feature_article
# - blog.archive_article
```

## Assigning Permissions

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

# Get permission
permission = Permission.objects.get(
    codename='publish_article',
    content_type__app_label='blog'
)

# Assign to user
user.user_permissions.add(permission)

# Remove from user
user.user_permissions.remove(permission)

# Clear all permissions
user.user_permissions.clear()

# Check (must refetch user or clear permission cache)
user = User.objects.get(pk=user.pk)  # Refetch
user.has_perm('blog.publish_article')
```

## Creating Permissions Programmatically

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(Article)

permission, created = Permission.objects.get_or_create(
    codename='export_article',
    name='Can export articles',
    content_type=content_type
)
```

## Customizing Default Permissions

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    
    class Meta:
        # Only create add and view permissions
        default_permissions = ('add', 'view')
        
        # Additional custom permissions
        permissions = [
            ('publish_article', 'Can publish'),
        ]
```

## Common Pitfalls

1. **Forgetting that superusers bypass all permission checks**: `has_perm()` always returns `True` for superusers, even for permissions that do not exist. This can mask bugs in your permission logic during development.
2. **Not refetching the user after assigning permissions**: Django caches permissions on the user object. After `user.user_permissions.add(perm)`, you must refetch the user or delete `user._perm_cache` to see the change.
3. **Using `permission_required` without `raise_exception=True`**: By default, the decorator redirects to the login page -- even for already-authenticated users who lack the permission. Set `raise_exception=True` to return a 403 instead.

## Best Practices

1. **Use `permission_required` decorator or `PermissionRequiredMixin`**: Declarative access control is easier to audit than manual `has_perm()` checks scattered in view logic.
2. **Assign permissions via groups, not directly to users**: This makes permission management scalable -- change the group's permissions once and all members are updated.
3. **Customize `default_permissions` for models that do not need all four**: Set `default_permissions = ('view',)` on read-only models to avoid creating unused permissions.

## Summary

- Django creates add, change, delete, and view permissions for every model automatically.
- Check permissions with `user.has_perm('app.codename')` or the `@permission_required` decorator.
- Custom permissions are defined in `Meta.permissions` and created by migrations.
- Assign permissions to groups, not individual users, for maintainability.
- Use the `perms` template variable for conditional display in templates.

## Code Examples

**Using Django's permission_required decorator to protect views**

```python
from django.contrib.auth.decorators import permission_required, login_required

@login_required
@permission_required('blog.add_article', raise_exception=True)
def create_article(request):
    # Only users with 'blog.add_article' permission can access
    pass
```


## Resources

- [Permissions and Authorization](https://docs.djangoproject.com/en/6.0/topics/auth/default/#permissions-and-authorization) — Official permissions documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*