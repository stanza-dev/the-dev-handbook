---
source_course: "django-authentication"
source_lesson: "django-authentication-groups-basics"
---

# Working with Django Groups

## Introduction

Django Groups let you bundle permissions together and assign them to users in bulk. This is the foundation of role-based access control (RBAC) -- the pattern used by virtually every production application to manage who can do what.

## Key Concepts

**Group**: A named set of permissions that can be assigned to multiple users.

**`user.groups`**: ManyToMany manager for accessing and modifying a user's group memberships.

**`group.permissions`**: ManyToMany manager for the permissions assigned to a group.

**`group_required` decorator**: Custom decorator pattern for restricting views to group members.

**`UserPassesTestMixin`**: Django's built-in mixin for custom access checks in class-based views.

## Real World Context

A school management system uses groups for Teachers, Students, and Administrators. When a teacher is promoted to department head, adding them to the Administrators group instantly grants access to grade reports, budget tools, and staff management -- without touching individual permissions.

## Deep Dive

Groups allow you to assign permissions to multiple users at once, implementing role-based access control (RBAC).

## Creating Groups

```python
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType


# Create groups programmatically
def create_groups():
    # Editor group
    editors, created = Group.objects.get_or_create(name='Editors')
    
    # Get content type for Article model
    article_ct = ContentType.objects.get_for_model(Article)
    
    # Add permissions to group
    permissions = Permission.objects.filter(
        content_type=article_ct,
        codename__in=['add_article', 'change_article', 'view_article']
    )
    editors.permissions.set(permissions)
    
    # Admin group with all article permissions
    admins, created = Group.objects.get_or_create(name='Admins')
    all_perms = Permission.objects.filter(content_type=article_ct)
    admins.permissions.set(all_perms)
    
    # Viewer group
    viewers, created = Group.objects.get_or_create(name='Viewers')
    view_perm = Permission.objects.get(
        content_type=article_ct,
        codename='view_article'
    )
    viewers.permissions.add(view_perm)
```

## Managing User Groups

```python
from django.contrib.auth.models import User, Group

# Add user to group
user = User.objects.get(username='john')
editors = Group.objects.get(name='Editors')
user.groups.add(editors)

# Add to multiple groups
user.groups.add(editors, viewers)

# Remove from group
user.groups.remove(editors)

# Set groups (replace existing)
user.groups.set([editors, viewers])

# Clear all groups
user.groups.clear()

# Check if user in group
if user.groups.filter(name='Editors').exists():
    print('User is an editor')

# Get all users in a group
editor_users = User.objects.filter(groups__name='Editors')
```

## Checking Group Membership in Views

```python
def dashboard_view(request):
    user = request.user
    
    # Check group membership
    is_editor = user.groups.filter(name='Editors').exists()
    is_admin = user.groups.filter(name='Admins').exists()
    
    # Get all user's group names
    user_groups = list(user.groups.values_list('name', flat=True))
    
    context = {
        'is_editor': is_editor,
        'is_admin': is_admin,
        'groups': user_groups,
    }
    return render(request, 'dashboard.html', context)
```

## Group-Based Decorators

```python
from django.contrib.auth.decorators import user_passes_test
from functools import wraps


def group_required(*group_names):
    """Require user to be in one of the specified groups."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if request.user.is_authenticated:
                if request.user.groups.filter(name__in=group_names).exists():
                    return view_func(request, *args, **kwargs)
            return redirect('permission_denied')
        return wrapper
    return decorator


# Usage
@group_required('Editors', 'Admins')
def edit_article(request, pk):
    # Only editors and admins can access
    pass


# Alternative with user_passes_test
@user_passes_test(lambda u: u.groups.filter(name='Admins').exists())
def admin_only_view(request):
    pass
```

## Group-Based Mixin for CBVs

```python
from django.contrib.auth.mixins import UserPassesTestMixin


class GroupRequiredMixin(UserPassesTestMixin):
    """Mixin requiring user to be in specified groups."""
    required_groups = []
    
    def test_func(self):
        if not self.request.user.is_authenticated:
            return False
        return self.request.user.groups.filter(
            name__in=self.required_groups
        ).exists()


class ArticleEditView(GroupRequiredMixin, UpdateView):
    required_groups = ['Editors', 'Admins']
    model = Article
    template_name = 'article_edit.html'
```

## Template Group Checks

```html
{% if request.user.groups.all %}
    <p>Your groups: 
    {% for group in request.user.groups.all %}
        {{ group.name }}{% if not forloop.last %}, {% endif %}
    {% endfor %}
    </p>
{% endif %}

<!-- Using a custom context processor or template tag -->
{% if 'Editors' in user_groups %}
    <a href="{% url 'article_edit' article.pk %}">Edit Article</a>
{% endif %}
```

## Custom Template Tag for Groups

```python
# templatetags/group_tags.py
from django import template

register = template.Library()


@register.filter
def in_group(user, group_name):
    """Check if user is in a specific group."""
    return user.groups.filter(name=group_name).exists()


@register.filter
def in_any_group(user, group_names):
    """Check if user is in any of the specified groups."""
    groups = [g.strip() for g in group_names.split(',')]
    return user.groups.filter(name__in=groups).exists()
```

```html
{% load group_tags %}

{% if user|in_group:'Editors' %}
    <a href="{% url 'article_edit' article.pk %}">Edit</a>
{% endif %}

{% if user|in_any_group:'Editors,Admins' %}
    <a href="{% url 'admin_panel' %}">Admin Panel</a>
{% endif %}
```

## Common Pitfalls

1. **Checking group membership with `user.groups.all()` instead of `filter().exists()`**: Fetching all groups into Python and iterating is slower and less readable than a single database-level `EXISTS` query.
2. **Hardcoding group names as strings**: If someone renames the "Editors" group to "Content Editors" in the admin, all `filter(name='Editors')` checks silently fail. Define group names as constants.
3. **Forgetting that superusers bypass group checks**: `has_perm()` returns True for superusers regardless of group membership. If your business logic depends on group membership (not just permissions), always check groups explicitly.

## Best Practices

1. **Define group names as constants**: `EDITOR_GROUP = 'Editors'` in a central module prevents typos and makes renames easy.
2. **Create a reusable `group_required` decorator**: Encapsulate the `filter(name__in=groups).exists()` pattern so views stay clean.
3. **Use management commands for group setup**: Run `python manage.py setup_groups` during deployment to ensure groups and their permissions are consistent across environments.

## Summary

- Groups bundle permissions for role-based access control.
- Use `user.groups.add(group)` and `user.groups.remove(group)` to manage memberships.
- Check membership with `user.groups.filter(name='X').exists()` for efficiency.
- Automate group creation with management commands for consistency.
- Create a `group_required` decorator for clean, reusable view protection.

## Code Examples

**Creating groups, assigning permissions, and adding users to groups**

```python
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType

# Create group
editors, created = Group.objects.get_or_create(name='Editors')

# Add permissions
article_ct = ContentType.objects.get_for_model(Article)
perms = Permission.objects.filter(
    content_type=article_ct,
    codename__in=['add_article', 'change_article', 'view_article']
)
editors.permissions.set(perms)

# Add user to group
user.groups.add(editors)
```


## Resources

- [Groups](https://docs.djangoproject.com/en/6.0/topics/auth/default/#groups) — Django groups documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*