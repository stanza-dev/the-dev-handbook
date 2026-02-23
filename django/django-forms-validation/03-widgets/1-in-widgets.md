---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-built-in-widgets"
---

## Introduction

Widgets control the HTML Django renders for form fields, handling presentation while fields handle validation.

## Key Concepts

- **Widget**: Renders HTML input and extracts POST data.
- **attrs**: Dict of HTML attributes for the widget.
- **TextInput / Textarea / PasswordInput**: Text widgets.
- **Select / RadioSelect**: Choice widgets.

## Real World Context

CSS framework integration uses `attrs` to add classes like `form-control`. Add `aria-label` for accessibility.

## Deep Dive

# Built-in Widgets

Widgets control the HTML rendering of form fields. Django provides widgets for all common input types.

## Text Widgets

```python
from django import forms

class TextWidgetsForm(forms.Form):
    # Default text input
    name = forms.CharField(widget=forms.TextInput)
    
    # With HTML attributes
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'your@email.com',
            'autocomplete': 'email',
        })
    )
    
    # Textarea
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'rows': 5,
            'cols': 40,
            'class': 'form-control',
        })
    )
    
    # Password (masked input)
    password = forms.CharField(
        widget=forms.PasswordInput(attrs={
            'autocomplete': 'new-password',
        })
    )
    
    # Hidden field
    csrf_token = forms.CharField(widget=forms.HiddenInput)
    
    # URL input
    website = forms.URLField(widget=forms.URLInput)
    
    # Number input
    quantity = forms.IntegerField(
        widget=forms.NumberInput(attrs={'min': 1, 'max': 100})
    )
```

## Choice Widgets

```python
class ChoiceWidgetsForm(forms.Form):
    CHOICES = [('a', 'Option A'), ('b', 'Option B'), ('c', 'Option C')]
    
    # Dropdown
    dropdown = forms.ChoiceField(
        choices=CHOICES,
        widget=forms.Select(attrs={'class': 'form-select'})
    )
    
    # Radio buttons
    radio = forms.ChoiceField(
        choices=CHOICES,
        widget=forms.RadioSelect
    )
    
    # Multiple select dropdown
    multi_select = forms.MultipleChoiceField(
        choices=CHOICES,
        widget=forms.SelectMultiple
    )
    
    # Checkboxes
    checkboxes = forms.MultipleChoiceField(
        choices=CHOICES,
        widget=forms.CheckboxSelectMultiple
    )
```

## Date and Time Widgets

```python
class DateWidgetsForm(forms.Form):
    # Date picker
    date = forms.DateField(
        widget=forms.DateInput(attrs={
            'type': 'date',
            'class': 'form-control',
        })
    )
    
    # Time picker
    time = forms.TimeField(
        widget=forms.TimeInput(attrs={
            'type': 'time',
        })
    )
    
    # DateTime picker
    datetime = forms.DateTimeField(
        widget=forms.DateTimeInput(attrs={
            'type': 'datetime-local',
        })
    )
    
    # Split date/time (separate inputs)
    split_datetime = forms.SplitDateTimeField(
        widget=forms.SplitDateTimeWidget(attrs={
            'date': {'type': 'date', 'class': 'date-input'},
            'time': {'type': 'time', 'class': 'time-input'},
        })
    )
```

## Boolean and File Widgets

```python
class MixedWidgetsForm(forms.Form):
    # Checkbox
    agree = forms.BooleanField(
        widget=forms.CheckboxInput(attrs={'class': 'form-check-input'})
    )
    
    # File input
    document = forms.FileField(
        widget=forms.FileInput(attrs={
            'accept': '.pdf,.doc,.docx',
            'class': 'form-control',
        })
    )
    
    # Clear file (for updates)
    photo = forms.ImageField(
        widget=forms.ClearableFileInput(attrs={
            'accept': 'image/*',
        })
    )
    
    # Multiple files
    attachments = forms.FileField(
        widget=forms.FileInput(attrs={'multiple': True})
    )
```

## Widget Attributes

```python
class AttributesForm(forms.Form):
    # Set attributes in widget
    field1 = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'id': 'custom-id',
            'data-validate': 'true',
            'aria-label': 'Field description',
        })
    )

# Or modify existing widget
class ModifiedForm(forms.Form):
    email = forms.EmailField()
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['email'].widget.attrs.update({
            'class': 'form-control',
            'placeholder': 'Enter email',
        })
```

## Rendering Widgets in Templates

```html
<!-- Widget types -->
{{ form.name }}  <!-- <input type="text" name="name" ...> -->
{{ form.email }}  <!-- <input type="email" name="email" ...> -->
{{ form.password }}  <!-- <input type="password" name="password" ...> -->

<!-- Access widget properties -->
{{ form.name.id_for_label }}  <!-- id_name -->
{{ form.name.html_name }}  <!-- name -->
{{ form.name.value }}  <!-- current value -->
```

## Common Pitfalls

1. **Wrong widget on EmailField** -- Use `EmailInput` not `TextInput`.
2. **Shared attrs at class level** -- Copy in `__init__`.
3. **DateInput defaults to text** -- Add `type=date` explicitly.

## Best Practices

1. **Loop attrs in `__init__`** -- Uniform CSS classes.
2. **Match widget to field** -- Correct mobile keyboard.
3. **Add `autocomplete`** -- Better browser autofill.

## Summary

- Widgets render HTML; fields validate.
- Pass attrs for CSS/HTML attributes.
- Use correct widget subclass for HTML5 types.
- Choice widgets: dropdown, radio, checkbox.
- Add autocomplete for UX.

## Code Examples

**Widgets control the HTML output of form fields -- use attrs to add CSS classes, placeholders, and other HTML attributes**

```python
from django import forms

class StyledForm(forms.Form):
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'your@email.com',
            'autocomplete': 'email',
        })
    )
    priority = forms.ChoiceField(
        choices=[('low', 'Low'), ('medium', 'Medium'), ('high', 'High')],
        widget=forms.RadioSelect,
    )
# email renders: <input type="email" class="form-control" placeholder="your@email.com">
# priority renders: <ul> with <input type="radio"> elements
```


## Resources

- [Built-in Widgets](https://docs.djangoproject.com/en/6.0/ref/forms/widgets/) — Complete widgets reference

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*