---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-inline-permissions"
---

# Inline Permissions and Customization

## Introduction

Control who can add, edit, or delete inline objects.

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

## Summary

Override permission methods to control access. Use get_queryset for filtering. Use custom formsets for validation.

## Resources

- [Inline Permissions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#inlinemodeladmin-options) — Inline options

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*