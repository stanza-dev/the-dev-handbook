---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-modelform-advanced"
---

# Advanced ModelForm Patterns

## Introduction

Beyond the basics, ModelForms support inheritance, multiple models in one view, and factory functions that reduce boilerplate. These patterns are essential for larger Django applications.

## Key Concepts

- **Form inheritance**: One ModelForm can extend another, sharing fields and validation.
- **Multiple forms per view**: Handle two or more ModelForms in a single request.
- **Factory functions**: Generate ModelForm classes dynamically with modelform_factory().

## Real World Context

In a real application you might have a UserForm and a ProfileForm that both appear on the same settings page. Or you might need an admin version of a form that includes extra fields. Inheritance and factory functions prevent code duplication across these scenarios.

## Deep Dive

### ModelForm Inheritance

ModelForms can inherit from each other. The child form gets all parent fields plus its own:

```python
from django import forms
from .models import Article


class BaseArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body']
        widgets = {
            'body': forms.Textarea(attrs={'rows': 10}),
        }


class AdminArticleForm(BaseArticleForm):
    """Extends the base form with admin-only fields."""
    class Meta(BaseArticleForm.Meta):
        fields = BaseArticleForm.Meta.fields + ['status', 'featured']
```

The child inherits widgets, labels, and validation from the parent while adding its own fields.

### Multiple ModelForms in One View

Use the prefix parameter to avoid field name collisions:

```python
from .models import User, Profile
from .forms import UserForm, ProfileForm


def settings_view(request):
    user = request.user
    profile = user.profile

    if request.method == 'POST':
        user_form = UserForm(
            request.POST, instance=user, prefix='user'
        )
        profile_form = ProfileForm(
            request.POST, request.FILES,
            instance=profile, prefix='profile'
        )

        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            return redirect('settings')
    else:
        user_form = UserForm(instance=user, prefix='user')
        profile_form = ProfileForm(
            instance=profile, prefix='profile'
        )

    return render(request, 'settings.html', {
        'user_form': user_form,
        'profile_form': profile_form,
    })
```

The prefix ensures each form's field names are unique in the HTML (e.g., user-email vs profile-bio).

### modelform_factory()

Generate a ModelForm class on the fly without writing a class definition:

```python
from django.forms import modelform_factory
from .models import Article

# Quick one-liner ModelForm
ArticleForm = modelform_factory(
    Article,
    fields=['title', 'body', 'status'],
    widgets={'body': forms.Textarea(attrs={'rows': 8})},
)

# Useful for generic views or admin customization
def get_form_class(model, user):
    if user.is_staff:
        return modelform_factory(model, fields='__all__')
    return modelform_factory(
        model, fields=['title', 'body']
    )
```

This is especially handy in generic views or when you need slight variations of a form.

## Common Pitfalls

1. **Forgetting prefix** — Without prefix, two forms with a 'name' field will clash and corrupt each other's data on POST.
2. **Meta inheritance gotcha** — The child Meta must explicitly inherit from the parent's Meta class (Meta(ParentForm.Meta)) or you lose the parent's configuration.

## Best Practices

1. **Use prefix for multi-form views** — Always set prefix when rendering more than one form on a page. It prevents subtle data corruption bugs.
2. **Prefer explicit form classes over factories** — Use modelform_factory for quick prototypes, but create proper classes for production forms that need custom validation.

## Summary

- ModelForm inheritance shares fields, widgets, and validation between form classes.
- Use prefix='...' when placing multiple forms on one page.
- modelform_factory() creates ModelForm classes dynamically without writing a class.
- Always inherit Meta from the parent when extending a ModelForm.

## Resources

- [ModelForm Factory](https://docs.djangoproject.com/en/6.0/ref/forms/models/#modelform-factory-function) — modelform_factory() reference documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*