---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-file-storage"
---

# File Storage Configuration

## Introduction

Configure where and how Django stores uploaded files.

## Deep Dive

### Settings

```python
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# For development - serve media files
if DEBUG:
    urlpatterns += static(MEDIA_URL, document_root=MEDIA_ROOT)
```

### Model File Fields

```python
class Document(models.Model):
    file = models.FileField(upload_to='documents/')
    image = models.ImageField(upload_to='images/%Y/%m/')
    
    # Dynamic upload path
    def user_directory_path(instance, filename):
        return f'user_{instance.user.id}/{filename}'
    
    avatar = models.ImageField(upload_to=user_directory_path)
```

### Accessing Files

```python
doc = Document.objects.get(pk=1)
print(doc.file.name)  # 'documents/report.pdf'
print(doc.file.url)   # '/media/documents/report.pdf'
print(doc.file.path)  # '/app/media/documents/report.pdf'
```

## Summary

Configure MEDIA_ROOT for storage location. Use upload_to for organization. Remember to serve media in development.

## Resources

- [File Storage](https://docs.djangoproject.com/en/6.0/topics/files/) — File handling

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*