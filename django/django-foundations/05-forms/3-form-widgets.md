---
source_course: "django-foundations"
source_lesson: "django-foundations-form-widgets"
---

# Form Widgets and Customization

## Introduction

Django widgets control how form fields are rendered as HTML. By customizing widgets, you can transform a plain text input into a date picker, a rich text editor, or a styled component that matches your design system.

## Key Concepts

- **Widget**: A Django class that renders a form field as an HTML input element.
- **attrs**: A dictionary of HTML attributes passed to the widget for styling and behavior.
- **Custom Widget**: A user-defined widget class for specialized rendering.

## Real World Context

Default Django form rendering produces functional but unstyled HTML. In real projects, you need forms that integrate with CSS frameworks like Bootstrap or Tailwind, use date pickers, and provide a polished user experience.

## Deep Dive

### Built-in Widgets

Django provides widgets for every HTML input type:

```python
from django import forms

class ArticleForm(forms.Form):
    # Text inputs
    title = forms.CharField(widget=forms.TextInput)
    body = forms.CharField(widget=forms.Textarea)
    slug = forms.CharField(widget=forms.HiddenInput)
    
    # Selection widgets
    category = forms.ChoiceField(
        choices=[('tech', 'Technology'), ('science', 'Science')],
        widget=forms.Select
    )
    tags = forms.MultipleChoiceField(
        choices=[('python', 'Python'), ('django', 'Django')],
        widget=forms.CheckboxSelectMultiple
    )
    
    # Date/time widgets
    publish_date = forms.DateField(widget=forms.DateInput(attrs={'type': 'date'}))
    publish_time = forms.TimeField(widget=forms.TimeInput(attrs={'type': 'time'}))
    
    # Password
    secret = forms.CharField(widget=forms.PasswordInput)
```

### Adding HTML Attributes

Use `attrs` to add CSS classes, placeholders, and other HTML attributes:

```python
class StyledContactForm(forms.Form):
    name = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Your full name',
            'id': 'contact-name',
            'autofocus': True,
        })
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'you@example.com',
        })
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
            'placeholder': 'Your message...',
        })
    )
```

### Customizing Widgets in ModelForms

Override widgets in the Meta class:

```python
class QuestionForm(forms.ModelForm):
    class Meta:
        model = Question
        fields = ['question_text', 'pub_date', 'is_active']
        widgets = {
            'question_text': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Enter your question',
            }),
            'pub_date': forms.DateTimeInput(attrs={
                'type': 'datetime-local',
                'class': 'form-control',
            }),
            'is_active': forms.CheckboxInput(attrs={
                'class': 'form-check-input',
            }),
        }
```

### Widget Reference

| Widget | HTML Output | Use Case |
|--------|-------------|----------|
| `TextInput` | `<input type="text">` | Short text |
| `Textarea` | `<textarea>` | Long text |
| `EmailInput` | `<input type="email">` | Email addresses |
| `URLInput` | `<input type="url">` | URLs |
| `NumberInput` | `<input type="number">` | Numbers |
| `PasswordInput` | `<input type="password">` | Passwords |
| `HiddenInput` | `<input type="hidden">` | Hidden values |
| `DateInput` | `<input type="text">` | Dates |
| `Select` | `<select>` | Dropdowns |
| `RadioSelect` | `<input type="radio">` | Radio buttons |
| `CheckboxInput` | `<input type="checkbox">` | Single checkbox |
| `CheckboxSelectMultiple` | Multiple checkboxes | Multi-select |
| `FileInput` | `<input type="file">` | File uploads |

### Rendering Individual Fields

For full control, render fields manually in templates:

```html
<form method="post">
    {% csrf_token %}
    <div class="mb-3">
        <label for="{{ form.name.id_for_label }}" class="form-label">
            {{ form.name.label }}
        </label>
        {{ form.name }}
        {% if form.name.help_text %}
            <small class="text-muted">{{ form.name.help_text }}</small>
        {% endif %}
        {% for error in form.name.errors %}
            <div class="text-danger">{{ error }}</div>
        {% endfor %}
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

## Common Pitfalls

- **Setting `type='date'` without `DateInput` widget**: The `attrs={'type': 'date'}` must be applied to a `DateInput` widget, not a generic `TextInput`, for proper date parsing.
- **Forgetting `render_value=False` on PasswordInput**: By default, `PasswordInput` does not re-render the password on validation errors. This is intentional for security.
- **Using `CheckboxSelectMultiple` without a list field**: This widget only works with fields that accept multiple values, like `MultipleChoiceField` or `TypedMultipleChoiceField`.

## Best Practices

- **Use the `widgets` dictionary in Meta** for ModelForms instead of overriding `__init__` to set widget attributes.
- **Create a form mixin** for consistent styling across all forms in your project.
- **Use HTML5 input types** (date, email, number, url) via widget attrs for built-in browser validation.

## Summary

- Widgets control how form fields are rendered as HTML elements
- Use `attrs` to add CSS classes, placeholders, IDs, and HTML5 attributes
- Common widgets include `TextInput`, `Textarea`, `Select`, `RadioSelect`, `CheckboxSelectMultiple`, and `DateInput`
- Customize ModelForm widgets in the `Meta.widgets` dictionary
- Render fields manually in templates for full control over styling and layout

## Code Examples

**Form with styled widgets using CSS class attributes and placeholders**

```python
from django import forms

class StyledContactForm(forms.Form):
    name = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Your full name',
        })
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'you@example.com',
        })
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
        })
    )
```


## Resources

- [Widgets](https://docs.djangoproject.com/en/6.0/ref/forms/widgets/) — Official reference for Django form widgets

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*