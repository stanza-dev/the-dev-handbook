---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-reusable-validators"
---

# Reusable Validators

## Introduction

Create validator functions and classes that can be reused across multiple fields and forms.

## Key Concepts

**Validator Function**: Callable that raises ValidationError.

**Validator Class**: Configurable validator with __call__.

## Deep Dive

### Simple Validator Function

```python
from django.core.exceptions import ValidationError

def validate_no_profanity(value):
    bad_words = ['spam', 'scam', 'xxx']
    for word in bad_words:
        if word in value.lower():
            raise ValidationError(f'Content contains prohibited word: {word}')

class CommentForm(forms.Form):
    content = forms.CharField(validators=[validate_no_profanity])
```

### Configurable Validator Class

```python
class FileSizeValidator:
    def __init__(self, max_mb=5):
        self.max_mb = max_mb
        self.max_bytes = max_mb * 1024 * 1024
    
    def __call__(self, file):
        if file.size > self.max_bytes:
            raise ValidationError(f'File must be under {self.max_mb}MB')

class UploadForm(forms.Form):
    document = forms.FileField(validators=[FileSizeValidator(max_mb=10)])
```

### Built-in Validators

```python
from django.core.validators import (
    MinLengthValidator, MaxLengthValidator,
    MinValueValidator, MaxValueValidator,
    RegexValidator, EmailValidator,
)

phone_validator = RegexValidator(
    regex=r'^\+?1?\d{9,15}$',
    message='Enter a valid phone number'
)
```

## Summary

Validator functions are simplest. Validator classes allow configuration. Use built-in validators when possible.

## Resources

- [Validators](https://docs.djangoproject.com/en/6.0/ref/validators/) — Built-in validators

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*