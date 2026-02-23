---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-modelform-validation"
---

# ModelForm Validation

## Introduction

ModelForms inherit all of Django's form validation plus add model-specific checks like unique constraints. Understanding how to customize validation on ModelForms lets you enforce complex business rules while leveraging the framework's built-in protections.

## Key Concepts

- **clean() override**: Custom form-wide validation that runs after field-level checks.
- **Unique constraints**: ModelForm automatically validates unique, unique_together, and UniqueConstraint.
- **validate_unique()**: Called during ModelForm validation to check uniqueness against the database.

## Real World Context

In production applications, you often need validation that goes beyond what the model definition alone can express. For example, ensuring a blog post slug is unique within its category, or that an event's end date is after its start date. ModelForm's clean() method is where this logic lives.

## Deep Dive

### Overriding clean() on ModelForm

When you override clean(), always call super().clean() first. This ensures that unique constraint validation still runs:

```python
from django import forms
from django.core.exceptions import ValidationError
from .models import Event


class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = ['title', 'start_date', 'end_date', 'venue']

    def clean(self):
        # super().clean() validates unique constraints
        cleaned_data = super().clean()
        start = cleaned_data.get('start_date')
        end = cleaned_data.get('end_date')

        if start and end and end <= start:
            raise ValidationError(
                'End date must be after the start date.'
            )

        return cleaned_data
```

The call to super().clean() is critical because it sets internal flags that trigger uniqueness validation against the database.

### Handling Unique Constraints

ModelForm automatically validates fields marked with `unique=True`, `unique_together`, and `UniqueConstraint`. You can customize error messages for these:

```python
from django.core.exceptions import NON_FIELD_ERRORS


class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'slug', 'category']
        error_messages = {
            NON_FIELD_ERRORS: {
                'unique_together': 'An article with this slug already '
                                   'exists in this category.',
            }
        }
```

This overrides the default uniqueness error message for the unique_together constraint.

### Custom Field Validation on ModelForm

Just like regular forms, you can define clean_<fieldname>() methods:

```python
class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['username', 'bio', 'website']

    def clean_username(self):
        username = self.cleaned_data['username']
        if username.startswith('admin'):
            raise ValidationError(
                'Usernames cannot start with "admin".'
            )
        return username.lower()
```

The returned value replaces what goes into cleaned_data, letting you normalize input.

## Common Pitfalls

1. **Forgetting super().clean()** — If you skip the super() call, unique constraint validation silently stops working. Always call it first.
2. **Excluding fields breaks uniqueness checks** — If a field involved in a unique_together constraint is excluded from the form, Django cannot validate that constraint and skips it silently.

## Best Practices

1. **Use error_messages in Meta** — Customize constraint violation messages to be user-friendly instead of showing raw database errors.
2. **Keep clean() focused** — Put single-field validation in clean_<fieldname>() and cross-field validation in clean(). This keeps the code organized and error messages correctly associated with fields.

## Summary

- Override clean() for cross-field validation; always call super().clean() first.
- ModelForm validates unique, unique_together, and UniqueConstraint automatically.
- Use clean_<fieldname>() for single-field custom validation with value normalization.
- Customize error messages via Meta.error_messages using NON_FIELD_ERRORS.

## Code Examples

**ModelForm with cross-field validation and unique constraint preservation via super().clean()**

```python
class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = ['title', 'start_date', 'end_date', 'venue']

    def clean(self):
        cleaned_data = super().clean()  # validates unique constraints
        start = cleaned_data.get('start_date')
        end = cleaned_data.get('end_date')

        if start and end and end <= start:
            raise ValidationError('End date must be after start date.')

        return cleaned_data
```


## Resources

- [ModelForm Validation](https://docs.djangoproject.com/en/6.0/topics/forms/modelforms/#overriding-the-clean-method) — Official guide to overriding clean() on ModelForms

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*