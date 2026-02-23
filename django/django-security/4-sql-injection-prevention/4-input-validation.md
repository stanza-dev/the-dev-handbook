---
source_course: "django-security"
source_lesson: "django-security-input-validation"
---

# Input Validation Best Practices

## Introduction

Validation prevents malicious data from reaching your database or being processed. Defense in depth means validating at multiple layers.

## Key Concepts

**Input Validation**: Checking that data conforms to expected format.

**Sanitization**: Cleaning data to remove dangerous content.

**Whitelisting**: Only allowing known-good values.



## Real World Context

Input validation failures have led to everything from XSS attacks (unexpected HTML in text fields) to denial-of-service (unbounded query parameters causing expensive database operations). Django forms are battle-tested validators that handle edge cases most custom code misses.

## Deep Dive

### Django Form Validation

```python
from django import forms
from django.core.validators import RegexValidator

class UserForm(forms.Form):
    username = forms.CharField(
        max_length=30,
        validators=[RegexValidator(r'^[a-zA-Z0-9_]+$', 'Alphanumeric only')]
    )
    email = forms.EmailField()
    age = forms.IntegerField(min_value=13, max_value=120)
    
    def clean_username(self):
        username = self.cleaned_data['username'].lower()
        if User.objects.filter(username=username).exists():
            raise forms.ValidationError('Username taken')
        return username
```

### Model Validation

```python
from django.core.exceptions import ValidationError
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    
    def clean(self):
        if self.status not in ['draft', 'published', 'archived']:
            raise ValidationError({'status': 'Invalid status'})
        if len(self.title) < 5:
            raise ValidationError({'title': 'Title too short'})
```

### Type Coercion Safety

```python
# DANGEROUS - trusting types
page = request.GET.get('page')  # Could be anything
Article.objects.all()[page:page+10]  # Crashes or weird behavior

# SAFE - validate and coerce
try:
    page = int(request.GET.get('page', 1))
    page = max(1, page)  # Ensure positive
except ValueError:
    page = 1
```



## Common Pitfalls

1. **Validating only on the client side** — JavaScript validation is a UX convenience, not a security control; attackers bypass it with curl or browser devtools.
2. **Using blacklists instead of whitelists** — Blocking known-bad patterns always misses something; define what is allowed and reject everything else.

## Best Practices

1. **Validate at entry points**: Forms, API endpoints, file uploads.
2. **Use Django's validators**: Built-in validators are battle-tested.
3. **Whitelist over blacklist**: Define what's allowed, not what's forbidden.
4. **Fail closed**: Reject invalid data rather than trying to fix it.

## Summary

Validation is your first defense. Use Django forms, model validation, and built-in validators. Always validate types, validate at entry points, and prefer whitelisting over blacklisting.

## Resources

- [Validators](https://docs.djangoproject.com/en/6.0/ref/validators/) — Django validators reference

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*