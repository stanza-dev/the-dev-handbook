---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-file-validation"
---

# File Validation

## Introduction

Validate file type, size, and content to prevent security issues.

## Key Concepts

- **FileExtensionValidator**: Checks extensions against allowed list.
- **Content-type validation**: Check actual content via python-magic.
- **File size validation**: Custom validator checking file.size.
- **clean_<field>()**: For image dimension validation.

## Real World Context

A document system must block executables disguised as PDFs. Combining extension and MIME type checking provides defense in depth.

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

## Common Pitfalls

1. **Trusting extensions alone** -- Renamed malware passes extension check.
2. **Not resetting file pointer** -- Call seek(0) after reading.
3. **InMemory vs Temporary uploads** -- Both expose .size but behave differently.

## Best Practices

1. **Layer multiple validators** -- Extension + MIME + size.
2. **Set DATA_UPLOAD_MAX_MEMORY_SIZE** -- Prevents DoS before validators run.

## Summary

Always validate file size and type. Don't trust file extensions alone - check actual content.

## Resources

- [File Validation](https://docs.djangoproject.com/en/6.0/ref/validators/#fileextensionvalidator) — File validation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*