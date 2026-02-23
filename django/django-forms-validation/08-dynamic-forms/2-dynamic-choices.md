---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-dynamic-choices"
---

# Dynamic Choices and Querysets

## Introduction

Form fields often need choices that depend on database state or the current user. Django's ModelChoiceField and dynamic choice assignment let you build forms whose options update at runtime.

## Key Concepts

- **ModelChoiceField**: A ChoiceField that populates its options from a queryset.
- **ModelMultipleChoiceField**: The multiple-select version of ModelChoiceField.
- **Dynamic queryset**: Overriding the queryset in __init__ to filter options per-request.

## Real World Context

Consider a task assignment form where the assignee dropdown should only show users in the current project. Or a product form where the category choices depend on the user's subscription tier. These scenarios require querysets that change based on runtime context.

## Deep Dive

### ModelChoiceField Basics

ModelChoiceField generates a select dropdown from a queryset:

```python
from django import forms
from .models import Category, Tag


class ArticleForm(forms.Form):
    title = forms.CharField(max_length=200)
    category = forms.ModelChoiceField(
        queryset=Category.objects.filter(active=True),
        empty_label='-- Select category --',
    )
    tags = forms.ModelMultipleChoiceField(
        queryset=Tag.objects.all(),
        widget=forms.CheckboxSelectMultiple,
        required=False,
    )
```

ModelChoiceField validates that the submitted value matches an object in the queryset, preventing users from submitting IDs they should not have access to.

### Dynamic Querysets in __init__

Filter choices based on the current user or request context:

```python
class TaskForm(forms.Form):
    title = forms.CharField(max_length=200)
    assignee = forms.ModelChoiceField(queryset=User.objects.none())
    project = forms.ModelChoiceField(queryset=Project.objects.none())

    def __init__(self, user, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Only show projects the user belongs to
        self.fields['project'].queryset = Project.objects.filter(
            members=user
        )
        # Only show team members
        self.fields['assignee'].queryset = User.objects.filter(
            projects__members=user
        ).distinct()
```

Starting with queryset=User.objects.none() is a safety pattern: if __init__ is called without a user, no options are available rather than exposing all users.

### Customizing the Display Label

By default, ModelChoiceField uses the model's __str__ method. Override label_from_instance for custom display:

```python
class UserChoiceField(forms.ModelChoiceField):
    def label_from_instance(self, obj):
        return f"{obj.get_full_name()} ({obj.email})"


class AssignmentForm(forms.Form):
    assignee = UserChoiceField(
        queryset=User.objects.filter(is_active=True)
    )
```

This shows "Jane Smith (jane@example.com)" instead of just the username.

### Dynamic Regular ChoiceField

For choices not backed by a model, set them in __init__:

```python
class ReportForm(forms.Form):
    report_type = forms.ChoiceField(choices=[])
    year = forms.ChoiceField(choices=[])

    def __init__(self, available_reports, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['report_type'].choices = [
            (r.slug, r.name) for r in available_reports
        ]
        current_year = 2026
        self.fields['year'].choices = [
            (y, str(y)) for y in range(current_year, current_year - 5, -1)
        ]
```

This keeps the form class reusable across different contexts.

## Common Pitfalls

1. **Stale querysets** — Defining queryset=Category.objects.all() at class level evaluates once at import time. If you add categories later, they won't appear. Use __init__ to set querysets dynamically or rely on Django's lazy evaluation.
2. **Security with ModelChoiceField** — Always scope the queryset to what the user should see. A queryset of User.objects.all() lets any user submit any user ID.

## Best Practices

1. **Start with .none()** — Initialize ModelChoiceField with queryset=Model.objects.none() and set the real queryset in __init__. This is a safe default if the form is instantiated without context.
2. **Override label_from_instance** — Instead of modifying __str__ on your model (which affects admin and shell too), create a custom field subclass with a tailored label.

## Summary

- ModelChoiceField populates options from a queryset and validates submitted values against it.
- Override querysets in __init__ for per-request filtering.
- Use label_from_instance() to customize how model objects are displayed in dropdowns.
- Start with .none() as a safe default for dynamic querysets.

## Resources

- [ModelChoiceField](https://docs.djangoproject.com/en/6.0/ref/forms/fields/#modelchoicefield) — ModelChoiceField API reference

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*