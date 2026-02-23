---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-generic-inlines"
---

# Generic Foreign Key Inlines

## Introduction

For models with GenericForeignKey, use GenericTabularInline or GenericStackedInline.

## Key Concepts

**GenericForeignKey**: A field that can point to any model via ContentType.

**GenericTabularInline**: Compact table inline for generic relations.

**GenericStackedInline**: Vertical layout inline for generic relations.

**ct_field / ct_fk_field**: Specify ContentType and object ID field names if they differ from defaults.

## Real World Context

A social platform uses a Comment model with a GenericForeignKey so the same comment system works on blog posts, photos, and videos. Using GenericTabularInline, admins can see and manage comments directly on any content type's change form without needing separate comment admin pages per content type.

## Deep Dive

### Generic Inline Setup

```python
from django.contrib.contenttypes.admin import GenericTabularInline

class CommentInline(GenericTabularInline):
    model = Comment
    extra = 0
    ct_field = 'content_type'  # ContentType field name
    ct_fk_field = 'object_id'   # Foreign key field name

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    inlines = [CommentInline]

@admin.register(Video)
class VideoAdmin(admin.ModelAdmin):
    inlines = [CommentInline]  # Same inline works for any model
```

### Comment Model

```python
from django.contrib.contenttypes.fields import GenericForeignKey
from django.contrib.contenttypes.models import ContentType

class Comment(models.Model):
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey('content_type', 'object_id')
    text = models.TextField()
```

## Common Pitfalls

1. **Mismatching ct_field and ct_fk_field names**: If your model uses non-default field names for the ContentType and object ID fields, you must explicitly set ct_field and ct_fk_field on the inline. Otherwise, Django raises a FieldError.

2. **Not indexing the content_type + object_id pair**: Generic relations without a composite index on (content_type_id, object_id) cause slow queries as the table grows.

## Best Practices

1. **Reuse generic inlines across admin classes**: A single CommentInline class can be added to the inlines of ArticleAdmin, PhotoAdmin, and VideoAdmin, keeping your admin code DRY.

2. **Set extra=0 on generic inlines**: Generic inlines often appear on many different admin pages. Keeping extra=0 avoids cluttering pages that may not need new objects.

## Summary

GenericTabularInline/GenericStackedInline work with GenericForeignKey. Specify ct_field and ct_fk_field if using non-default names.

## Resources

- [Generic Inlines](https://docs.djangoproject.com/en/6.0/ref/contrib/contenttypes/#generic-relations-in-admin) — Generic inline admin

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*