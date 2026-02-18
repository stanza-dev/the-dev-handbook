---
source_course: "django-authentication"
source_lesson: "django-authentication-custom-permissions"
---

# Custom Permissions

## Introduction

Beyond CRUD, apps need custom permissions like 'publish' or 'archive'.

## Key Concepts

**Meta.permissions**: Define in model class.

## Deep Dive

### Defining Custom Permissions

```python
class Article(models.Model):
    class Meta:
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature on homepage'),
        ]
```

### Using Custom Permissions

```python
@permission_required('blog.publish_article', raise_exception=True)
def publish(request, pk):
    article.status = 'published'
    article.save()
```

## Summary

Define custom permissions in Meta. Use permission_required decorator.

## Resources

- [Custom Permissions](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/#custom-permissions) — Custom permissions

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*