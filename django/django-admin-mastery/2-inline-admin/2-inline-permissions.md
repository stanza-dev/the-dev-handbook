---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-inline-permissions"
---

# Inline Permissions and Customization

## Introduction

Control who can add, edit, or delete inline objects.

## Key Concepts

**has_add_permission**: Controls whether new inline objects can be added.

**has_delete_permission**: Controls whether inline objects can be removed.

**has_change_permission**: Controls whether inline objects can be edited.

**get_queryset()**: Filters which inline objects are visible.

## Real World Context

In a project management tool, tasks belong to projects. Managers should be able to add and remove tasks, but regular team members should only be able to view and edit existing ones. By overriding has_add_permission and has_delete_permission on the TaskInline, you enforce these rules at the admin level without any custom middleware.

## Deep Dive

### Permission Methods

```python
class ChapterInline(admin.TabularInline):
    model = Chapter
    
    def has_add_permission(self, request, obj=None):
        # Only allow adding if book is not published
        if obj and obj.status == 'published':
            return False
        return super().has_add_permission(request, obj)
    
    def has_delete_permission(self, request, obj=None):
        return request.user.is_superuser
```

### Dynamic Queryset

```python
class ChapterInline(admin.TabularInline):
    model = Chapter
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        return qs.filter(is_hidden=False).order_by('order')
```

### Formset Customization

```python
from django.forms import BaseInlineFormSet

class ChapterFormSet(BaseInlineFormSet):
    def clean(self):
        super().clean()
        if len(self.forms) < 1:
            raise ValidationError('At least one chapter required')

class ChapterInline(admin.TabularInline):
    model = Chapter
    formset = ChapterFormSet
```

## Common Pitfalls

1. **Forgetting the obj parameter**: Inline permission methods receive the parent object as `obj`. Not using it means you cannot implement object-level rules like 'no changes to closed projects'.

2. **Not calling super()**: If you override a permission method without calling super(), you bypass Django's default permission checks, potentially granting unintended access.

## Best Practices

1. **Combine permission methods with get_queryset()**: Hiding objects via get_queryset() is not a security boundary. Always pair it with permission methods for proper access control.

2. **Use can_delete=False for audit trails**: When inline objects should never be deleted from the admin (e.g., payment records), set can_delete=False to remove the delete checkbox entirely.

## Summary

Override permission methods to control access. Use get_queryset for filtering. Use custom formsets for validation.

## Resources

- [Inline Permissions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#inlinemodeladmin-options) — Inline options

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*