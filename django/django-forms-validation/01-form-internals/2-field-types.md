---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-field-types"
---

# Field Types and Options

Django provides many field types for different kinds of input. Each has specific options and validation.

## Text Fields

```python
from django import forms

class TextForm(forms.Form):
    # Single line text
    name = forms.CharField(
        max_length=100,
        min_length=2,
        strip=True,  # Remove whitespace (default)
        empty_value='',  # Value if empty
    )
    
    # Multi-line text
    bio = forms.CharField(
        widget=forms.Textarea,
        max_length=500,
        required=False,
    )
    
    # Email with validation
    email = forms.EmailField()
    
    # URL with validation
    website = forms.URLField(required=False)
    
    # Slug (letters, numbers, underscores, hyphens)
    slug = forms.SlugField()
    
    # UUID
    uuid = forms.UUIDField()
    
    # Regex validated
    phone = forms.RegexField(
        regex=r'^\d{3}-\d{3}-\d{4}$',
        error_messages={'invalid': 'Format: XXX-XXX-XXXX'}
    )
```

## Number Fields

```python
class NumberForm(forms.Form):
    # Integer
    age = forms.IntegerField(
        min_value=0,
        max_value=150,
    )
    
    # Decimal (precise)
    price = forms.DecimalField(
        max_digits=10,
        decimal_places=2,
        min_value=0,
    )
    
    # Float (less precise)
    rating = forms.FloatField(
        min_value=0,
        max_value=5,
    )
```

## Date and Time Fields

```python
class DateTimeForm(forms.Form):
    # Date only
    birth_date = forms.DateField(
        widget=forms.DateInput(attrs={'type': 'date'}),
        input_formats=['%Y-%m-%d', '%m/%d/%Y'],
    )
    
    # Time only
    start_time = forms.TimeField(
        widget=forms.TimeInput(attrs={'type': 'time'}),
    )
    
    # Date and time
    event_datetime = forms.DateTimeField(
        widget=forms.DateTimeInput(attrs={'type': 'datetime-local'}),
    )
    
    # Duration
    duration = forms.DurationField(
        help_text='Format: DD HH:MM:SS',
    )
```

## Choice Fields

```python
class ChoiceForm(forms.Form):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    # Dropdown
    status = forms.ChoiceField(
        choices=STATUS_CHOICES,
        initial='draft',
    )
    
    # Radio buttons
    priority = forms.ChoiceField(
        choices=[('low', 'Low'), ('medium', 'Medium'), ('high', 'High')],
        widget=forms.RadioSelect,
    )
    
    # Multiple select
    categories = forms.MultipleChoiceField(
        choices=[('tech', 'Tech'), ('news', 'News'), ('sports', 'Sports')],
        widget=forms.CheckboxSelectMultiple,
    )
    
    # Typed choice (returns proper type)
    year = forms.TypedChoiceField(
        choices=[(y, y) for y in range(2020, 2030)],
        coerce=int,  # Convert to int
    )
```

## Boolean and File Fields

```python
class MixedForm(forms.Form):
    # Checkbox
    agree_terms = forms.BooleanField(
        required=True,
        label='I agree to the terms',
    )
    
    # Nullable boolean
    has_car = forms.NullBooleanField(
        widget=forms.Select(choices=[
            (None, 'Unknown'),
            (True, 'Yes'),
            (False, 'No'),
        ])
    )
    
    # File upload
    document = forms.FileField(
        required=False,
        allow_empty_file=False,
    )
    
    # Image upload (validates it's an image)
    avatar = forms.ImageField(
        required=False,
    )
```

## Common Field Options

```python
class OptionsForm(forms.Form):
    field = forms.CharField(
        # Validation
        required=True,           # Must have value
        validators=[validator],  # Custom validators
        
        # Display
        label='Field Label',     # HTML label
        label_suffix=':',        # After label
        help_text='Help text',   # Hint text
        
        # Initial value
        initial='default',       # Default value
        
        # HTML attributes
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Enter value',
        }),
        
        # Error messages
        error_messages={
            'required': 'This field is required.',
            'invalid': 'Enter a valid value.',
        },
        
        # Accessibility
        disabled=False,          # Read-only in HTML
    )
```

## Resources

- [Built-in Form Fields](https://docs.djangoproject.com/en/6.0/ref/forms/fields/) — Complete list of form field types

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*