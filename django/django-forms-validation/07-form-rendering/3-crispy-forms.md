---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-crispy-forms"
---

# Crispy Forms Integration

## Introduction

django-crispy-forms is a popular third-party library that lets you control form rendering through Python code rather than template markup. It supports Bootstrap, Tailwind CSS, and other CSS frameworks out of the box.

## Key Concepts

- **FormHelper**: A class that controls the form's layout, CSS classes, and submit button.
- **Layout**: Defines the visual structure of form fields using Python objects.
- **Template pack**: A set of templates for a specific CSS framework (bootstrap5, tailwind, etc.).

## Real World Context

Most Django projects use a CSS framework. Without crispy-forms, you end up writing repetitive template code to wrap each field in framework-specific div classes. Crispy-forms lets you define this once in Python and reuse it everywhere.

## Deep Dive

### Installation and Setup

Install the library and a template pack:

```bash
pip install django-crispy-forms crispy-bootstrap5
```

Configure in settings:

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'crispy_forms',
    'crispy_bootstrap5',
]

CRISPY_ALLOWED_TEMPLATE_PACKS = 'bootstrap5'
CRISPY_TEMPLATE_PACK = 'bootstrap5'
```

### Basic Usage with the Template Tag

The simplest approach uses the crispy template filter:

```html
{% load crispy_forms_tags %}

<form method="post">
    {% csrf_token %}
    {{ form|crispy }}
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

This renders every field with proper Bootstrap 5 classes automatically.

### Using FormHelper for Layout Control

For more control, attach a FormHelper to your form:

```python
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Layout, Submit, Row, Column, Field


class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    phone = forms.CharField(required=False)
    message = forms.CharField(widget=forms.Textarea)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.form_method = 'post'
        self.helper.form_class = 'form-horizontal'
        self.helper.layout = Layout(
            Row(
                Column('name', css_class='col-md-6'),
                Column('email', css_class='col-md-6'),
            ),
            'phone',
            Field('message', rows=5),
            Submit('submit', 'Send Message', css_class='btn-primary mt-3'),
        )
```

Render in the template with the crispy tag:

```html
{% load crispy_forms_tags %}
{% crispy form %}
```

The helper controls the entire form including the submit button, so no extra template markup is needed.

### Tailwind CSS with Crispy

For Tailwind, use the crispy-tailwind package:

```bash
pip install crispy-tailwind
```

```python
# settings.py
INSTALLED_APPS += ['crispy_tailwind']
CRISPY_ALLOWED_TEMPLATE_PACKS = 'tailwind'
CRISPY_TEMPLATE_PACK = 'tailwind'
```

The form renders with Tailwind utility classes automatically.

## Common Pitfalls

1. **Missing template pack** — Installing django-crispy-forms without a template pack (like crispy-bootstrap5) causes a TemplateDoesNotExist error. Always install both.
2. **Mixing crispy with manual rendering** — If you use FormHelper with a submit button, the template tag renders the full form including <form> tags. Don't wrap it in another <form> tag.

## Best Practices

1. **Use FormHelper for complex layouts** — Row/Column layout objects replace complex template markup with clean Python code.
2. **Set form_tag = False for partial forms** — When embedding a crispy form inside an existing form tag, set self.helper.form_tag = False to prevent nested form elements.

## Summary

- django-crispy-forms renders forms with CSS framework classes automatically.
- Use the |crispy filter for quick rendering or FormHelper for full layout control.
- Install a template pack matching your CSS framework (bootstrap5, tailwind).
- Layout objects like Row, Column, and Field give you grid control from Python.

## Resources

- [django-crispy-forms Documentation](https://django-crispy-forms.readthedocs.io/en/latest/) — Official crispy-forms documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*