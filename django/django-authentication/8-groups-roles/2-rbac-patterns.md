---
source_course: "django-authentication"
source_lesson: "django-authentication-rbac-patterns"
---

# Role-Based Access Control Patterns

## Introduction

Implement RBAC patterns for cleaner permission management.

## Key Concepts

**Role**: Named collection of permissions.

**RBAC**: Permission assignment through roles.

## Real World Context

In a media company CMS, the role hierarchy is clear: Viewers can read, Authors can write, Editors can publish, and Admins can delete. Mapping these roles to Django groups with inherited permissions means onboarding a new Author is a single `user.groups.add(authors)` call instead of assigning 15 individual permissions.

## Deep Dive

### Group-Based RBAC

```python
# Setup roles as groups
ROLES = {
    'viewer': ['view_article'],
    'editor': ['view_article', 'add_article', 'change_article'],
    'publisher': ['view_article', 'add_article', 'change_article', 'publish_article'],
    'admin': ['view_article', 'add_article', 'change_article', 'delete_article', 'publish_article'],
}

def setup_roles():
    for role_name, perms in ROLES.items():
        group, _ = Group.objects.get_or_create(name=role_name)
        for perm_codename in perms:
            perm = Permission.objects.get(codename=perm_codename)
            group.permissions.add(perm)
```

### Role Hierarchy

```python
class RoleChecker:
    HIERARCHY = {
        'admin': ['publisher', 'editor', 'viewer'],
        'publisher': ['editor', 'viewer'],
        'editor': ['viewer'],
        'viewer': [],
    }
    
    @classmethod
    def has_role(cls, user, required_role):
        user_roles = set(user.groups.values_list('name', flat=True))
        for role in user_roles:
            if role == required_role:
                return True
            if required_role in cls.HIERARCHY.get(role, []):
                return True
        return False
```

## Common Pitfalls

1. **Assigning permissions directly to users instead of groups**: Direct permissions become impossible to audit at scale. When an employee changes roles, you must manually remove each old permission instead of just switching groups.
2. **Not automating group setup**: If groups are created manually in the admin, they are inconsistent across environments. Use a management command or data migration to define roles as code.
3. **Overly complex hierarchies**: More than 4-5 role levels become hard to reason about. If your hierarchy is deeper, consider using a dedicated RBAC library like django-role-permissions.

## Best Practices

1. **Use groups for roles**: Don't assign permissions directly.
2. **Define role hierarchy**: Reduce permission duplication.

## Summary

Map roles to Django groups. Define clear role hierarchies. Assign permissions to groups, not users.

## Resources

- [RBAC](https://docs.djangoproject.com/en/6.0/topics/auth/default/#groups) — Groups for RBAC

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*