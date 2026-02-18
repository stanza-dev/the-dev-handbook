---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-generic-inlines"
---

# Generic Foreign Key Inlines

## Introduction

For models with GenericForeignKey, use GenericTabularInline or GenericStackedInline.

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

## Summary

GenericTabularInline/GenericStackedInline work with GenericForeignKey. Specify ct_field and ct_fk_field if using non-default names.

## Resources

- [Generic Inlines](https://docs.djangoproject.com/en/6.0/ref/contrib/contenttypes/#generic-relations-in-admin) — Generic inline admin

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*