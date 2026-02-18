---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-file-validation"
---

# File Validation

## Introduction

Validate file type, size, and content to prevent security issues.

## Deep Dive

### File Size Validator

```python
from django.core.exceptions import ValidationError

def validate_file_size(file):
    max_size = 5 * 1024 * 1024  # 5MB
    if file.size > max_size:
        raise ValidationError(f'File too large. Max size is 5MB.')

class UploadForm(forms.Form):
    document = forms.FileField(validators=[validate_file_size])
```

### File Extension Validation

```python
from django.core.validators import FileExtensionValidator

class UploadForm(forms.Form):
    document = forms.FileField(
        validators=[FileExtensionValidator(allowed_extensions=['pdf', 'docx'])]
    )
```

### Content Type Validation

```python
import magic  # pip install python-magic

def validate_file_type(file):
    mime = magic.from_buffer(file.read(1024), mime=True)
    file.seek(0)  # Reset file pointer
    
    allowed = ['application/pdf', 'image/jpeg', 'image/png']
    if mime not in allowed:
        raise ValidationError('Invalid file type')
```

## Summary

Always validate file size and type. Don't trust file extensions alone - check actual content.

## Resources

- [File Validation](https://docs.djangoproject.com/en/6.0/ref/validators/#fileextensionvalidator) — File validation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*