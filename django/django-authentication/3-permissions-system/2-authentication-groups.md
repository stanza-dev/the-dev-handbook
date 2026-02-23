---
source_course: "django-authentication"
source_lesson: "django-authentication-groups"
---

# Groups and Role-Based Access

## Introduction

Groups are Django's built-in mechanism for role-based access control. Instead of assigning permissions to individual users, you assign them to groups, making permission management scalable across hundreds of users.

## Key Concepts

**Group**: A named collection of permissions that can be assigned to multiple users.

**ManyToMany Relationship**: Users can belong to multiple groups, and groups can have multiple users.

**Group Permissions**: Any permission assigned to a group is automatically available to all its members.

**`user_passes_test`**: Decorator for custom access checks like group membership.

## Real World Context

A magazine publisher creates Editor, Writer, and Reviewer groups. When a freelancer joins, adding them to the Writer group grants exactly the right permissions. When they leave, removing them from the group revokes everything -- no risk of leftover permissions.

## Deep Dive

Groups let you assign permissions to multiple users at once, implementing role-based access control (RBAC).

## Creating Groups

```python
from django.contrib.auth.models import Group, Permission

# Create groups
editors = Group.objects.create(name='Editors')
authors = Group.objects.create(name='Authors')
moderators = Group.objects.create(name='Moderators')

# Add permissions to groups
edit_perms = Permission.objects.filter(
    codename__in=['add_article', 'change_article', 'delete_article', 'view_article']
)
editors.permissions.add(*edit_perms)

# Authors can only add and view
author_perms = Permission.objects.filter(
    codename__in=['add_article', 'view_article']
)
authors.permissions.add(*author_perms)
```

## Managing User Groups

```python
from django.contrib.auth.models import Group

# Add user to group
editors = Group.objects.get(name='Editors')
user.groups.add(editors)

# Remove from group
user.groups.remove(editors)

# Check group membership
if user.groups.filter(name='Editors').exists():
    print("User is an editor")

# Get all groups
for group in user.groups.all():
    print(group.name)

# Set multiple groups
user.groups.set([editors, authors])

# Clear all groups
user.groups.clear()
```

## Group Permissions Work Automatically

```python
# User is in 'Editors' group which has 'change_article' permission
user.has_perm('blog.change_article')  # True

# Get permissions from groups
user.get_group_permissions()
# {'blog.add_article', 'blog.change_article', ...}
```

## Group-Based View Access

```python
def group_required(*group_names):
    """Decorator to require group membership."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return redirect('login')
            
            if request.user.groups.filter(name__in=group_names).exists():
                return view_func(request, *args, **kwargs)
            
            return HttpResponseForbidden("You don't have access to this page.")
        return wrapper
    return decorator


@group_required('Editors', 'Admins')
def editor_dashboard(request):
    pass
```

## Hierarchical Roles

```python
# models.py
class Role(models.Model):
    ROLE_CHOICES = [
        (1, 'Viewer'),
        (2, 'Author'),
        (3, 'Editor'),
        (4, 'Admin'),
    ]
    
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    level = models.IntegerField(choices=ROLE_CHOICES, default=1)
    
    def has_role(self, required_level):
        """Check if user's role meets required level."""
        return self.level >= required_level


# Usage
if user.role.has_role(3):  # Editor or higher
    # Can edit
    pass
```

## Data Fixture for Groups

```python
# management/commands/setup_groups.py
from django.core.management.base import BaseCommand
from django.contrib.auth.models import Group, Permission


class Command(BaseCommand):
    help = 'Create default groups with permissions'
    
    def handle(self, *args, **options):
        # Define groups and their permissions
        groups_permissions = {
            'Viewers': [
                'view_article',
            ],
            'Authors': [
                'view_article',
                'add_article',
            ],
            'Editors': [
                'view_article',
                'add_article',
                'change_article',
                'publish_article',
            ],
            'Admins': [
                'view_article',
                'add_article',
                'change_article',
                'delete_article',
                'publish_article',
                'feature_article',
            ],
        }
        
        for group_name, permission_codenames in groups_permissions.items():
            group, created = Group.objects.get_or_create(name=group_name)
            
            for codename in permission_codenames:
                try:
                    permission = Permission.objects.get(codename=codename)
                    group.permissions.add(permission)
                except Permission.DoesNotExist:
                    self.stdout.write(f'Permission {codename} not found')
            
            action = 'Created' if created else 'Updated'
            self.stdout.write(f'{action} group: {group_name}')
```

Run with: `python manage.py setup_groups`

## Common Pitfalls

1. **Checking group membership with `user.groups.all()` in a loop**: This fires a database query per iteration. Use `user.groups.filter(name__in=[...]).exists()` for efficient batch checking.
2. **Creating groups manually in Django admin without automation**: Groups created in the admin are not version-controlled and may differ between environments. Use a management command or data migration.
3. **Assuming group names are case-insensitive**: Django group names are case-sensitive. `Editors` and `editors` are two different groups, which can cause subtle permission bugs.

## Best Practices

1. **Define groups in a management command**: Run it during deployment to ensure consistent groups across all environments.
2. **Use `get_or_create()` for idempotent group setup**: This prevents errors when the setup command runs multiple times.
3. **Create a `group_required` decorator**: Wrap the common `user.groups.filter(name__in=groups).exists()` check in a reusable decorator for cleaner views.

## Summary

- Groups are Django's built-in tool for role-based access control (RBAC).
- Assign permissions to groups, then add users to groups for scalable management.
- Use `user.groups.filter(name='X').exists()` for efficient membership checks.
- Automate group creation with management commands for consistency across environments.
- Group permissions combine with direct user permissions -- `has_perm()` checks both.

## Resources

- [Groups](https://docs.djangoproject.com/en/6.0/topics/auth/default/#groups) — Official documentation on Django groups

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*