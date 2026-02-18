---
source_course: "django-authentication"
source_lesson: "django-authentication-groups-basics"
---

# Working with Django Groups

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

## Resources

- [Groups](https://docs.djangoproject.com/en/6.0/topics/auth/default/#groups) — Django groups documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*