---
source_course: "django-authentication"
source_lesson: "django-authentication-object-permissions"
---

# Object-Level Permissions

## Introduction

Django's default permissions are model-level. For per-object permissions, use django-guardian or implement custom logic.

## Key Concepts

**Model Permissions**: Apply to all instances.

**Object Permissions**: Apply to specific instances.

## Real World Context

In a project management tool, an `Editor` permission on the Project model is not enough -- users should only edit *their own* projects. Object-level permissions let you check `user.has_perm('edit_project', project_instance)`, so a contributor cannot modify projects they were not invited to.

## Deep Dive

### Simple Object Permissions

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    
    def can_edit(self, user):
        if user.is_superuser:
            return True
        if user == self.author:
            return True
        return user.has_perm('blog.change_article')
```

### Using django-guardian

```python
# pip install django-guardian
from guardian.shortcuts import assign_perm, get_perms

# Assign permission to user for specific object
assign_perm('change_article', user, article)

# Check permission
user.has_perm('change_article', article)
```

## Common Pitfalls

1. **Assuming `has_perm()` checks object-level permissions automatically**: Without a backend like django-guardian, `user.has_perm('app.change_model', obj)` ignores the object argument and only checks model-level permissions.
2. **N+1 query problems**: Checking object permissions inside a loop (e.g., filtering a list of items) fires a database query per object. Use queryset-level filtering instead.
3. **Not falling back to model permissions**: Object-level checks should supplement, not replace, model-level permissions. A user with model-level `change_article` should still pass an object-level check.

## Best Practices

1. **Keep it simple**: Use model methods first.
2. **Cache permissions**: Object permissions can be slow.

## Summary

Model methods work for simple cases. Use django-guardian for complex scenarios.

## Resources

- [django-guardian](https://django-guardian.readthedocs.io/) — Object-level permissions

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*