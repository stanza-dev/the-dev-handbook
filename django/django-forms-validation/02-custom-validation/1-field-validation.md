---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-field-validation"
---

# Field-Level Validation

Validate individual fields with custom logic using clean_<fieldname> methods and validators.

## clean_<fieldname> Methods

```python
from django import forms
from django.core.exceptions import ValidationError


class RegistrationForm(forms.Form):
    username = forms.CharField(max_length=30)
    email = forms.EmailField()
    age = forms.IntegerField()
    
    def clean_username(self):
        """Validate username field."""
        username = self.cleaned_data['username']
        
        # Check for reserved names
        reserved = ['admin', 'root', 'system', 'moderator']
        if username.lower() in reserved:
            raise ValidationError('This username is reserved.')
        
        # Check for existing user
        if User.objects.filter(username__iexact=username).exists():
            raise ValidationError('Username already taken.')
        
        # Must return the cleaned value
        return username.lower()  # Normalize to lowercase
    
    def clean_email(self):
        """Validate email field."""
        email = self.cleaned_data['email']
        
        # Domain validation
        domain = email.split('@')[1]
        blocked_domains = ['tempmail.com', 'throwaway.com']
        if domain in blocked_domains:
            raise ValidationError('Please use a valid email address.')
        
        return email
    
    def clean_age(self):
        """Validate age field."""
        age = self.cleaned_data['age']
        
        if age < 13:
            raise ValidationError('You must be at least 13 years old.')
        
        return age
```

## Reusable Validators

Create validators that can be reused across forms:

```python
# validators.py
from django.core.exceptions import ValidationError
import re


def validate_no_special_chars(value):
    """Ensure value contains only letters and numbers."""
    if not re.match(r'^[a-zA-Z0-9_]+$', value):
        raise ValidationError(
            'Only letters, numbers, and underscores are allowed.',
            code='invalid_characters'
        )


def validate_no_profanity(value):
    """Check for inappropriate words."""
    profanity_list = ['badword1', 'badword2']  # Use a proper list
    for word in profanity_list:
        if word in value.lower():
            raise ValidationError(
                'Please avoid inappropriate language.',
                code='profanity'
            )


class MinWordsValidator:
    """Validator class for minimum word count."""
    
    def __init__(self, min_words=10):
        self.min_words = min_words
    
    def __call__(self, value):
        word_count = len(value.split())
        if word_count < self.min_words:
            raise ValidationError(
                f'Please write at least {self.min_words} words. '
                f'You have {word_count}.',
                code='min_words',
                params={'min_words': self.min_words, 'word_count': word_count}
            )


class FileExtensionValidator:
    """Validate file extensions."""
    
    def __init__(self, allowed_extensions):
        self.allowed_extensions = [ext.lower() for ext in allowed_extensions]
    
    def __call__(self, value):
        ext = value.name.split('.')[-1].lower()
        if ext not in self.allowed_extensions:
            raise ValidationError(
                f'File type not allowed. Allowed: {self.allowed_extensions}',
                code='invalid_extension'
            )
```

## Using Validators

```python
from .validators import (
    validate_no_special_chars,
    validate_no_profanity,
    MinWordsValidator,
    FileExtensionValidator
)


class ArticleForm(forms.Form):
    title = forms.CharField(
        max_length=200,
        validators=[validate_no_profanity]
    )
    
    slug = forms.SlugField(
        validators=[validate_no_special_chars]
    )
    
    content = forms.CharField(
        widget=forms.Textarea,
        validators=[
            MinWordsValidator(min_words=100),
            validate_no_profanity,
        ]
    )
    
    attachment = forms.FileField(
        required=False,
        validators=[FileExtensionValidator(['pdf', 'doc', 'docx'])]
    )
```

## Error Messages with Parameters

```python
def validate_length_range(min_len, max_len):
    """Factory function for length validator."""
    def validator(value):
        length = len(value)
        if length < min_len or length > max_len:
            raise ValidationError(
                'Length must be between %(min)d and %(max)d characters. '
                'Current length: %(current)d.',
                code='length_range',
                params={
                    'min': min_len,
                    'max': max_len,
                    'current': length
                }
            )
    return validator

# Usage
username = forms.CharField(
    validators=[validate_length_range(3, 20)]
)
```

## Resources

- [Form and Field Validation](https://docs.djangoproject.com/en/6.0/ref/forms/validation/) — Official validation documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*