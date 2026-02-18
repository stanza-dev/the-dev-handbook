---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-formset-validation"
---

# Formset Validation

## Introduction

Customize formset validation beyond individual form validation.

## Deep Dive

### Custom Formset Class

```python
from django.forms import BaseFormSet

class BaseBookFormSet(BaseFormSet):
    def clean(self):
        if any(self.errors):
            return  # Skip if form errors exist
        
        titles = []
        for form in self.forms:
            if form.cleaned_data:
                title = form.cleaned_data.get('title')
                if title in titles:
                    raise ValidationError('Duplicate titles not allowed')
                titles.append(title)

BookFormSet = formset_factory(
    BookForm,
    formset=BaseBookFormSet,
    extra=3
)
```

### Min/Max Forms

```python
BookFormSet = formset_factory(
    BookForm,
    min_num=1,      # Minimum forms required
    max_num=10,     # Maximum forms allowed
    validate_min=True,
    validate_max=True,
)
```

## Summary

Override clean() in BaseFormSet for cross-form validation. Use min_num/max_num to enforce limits.

## Resources

- [Formset Validation](https://docs.djangoproject.com/en/6.0/topics/forms/formsets/#custom-formset-validation) — Formset validation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*