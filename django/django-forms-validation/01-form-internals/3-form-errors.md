---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-form-errors"
---

# Working with Form Errors

## Introduction

Understanding how to access and display form errors is essential for good UX.

## Key Concepts

**Field Errors**: Errors specific to a field.

**Non-Field Errors**: Form-wide validation errors.

## Deep Dive

### Accessing Errors

```python
if not form.is_valid():
    # All errors as dict
    print(form.errors)  # {'email': ['Enter a valid email.']}
    
    # Specific field errors
    print(form.errors.get('email', []))
    
    # Non-field errors (from clean())
    print(form.non_field_errors())
    
    # As JSON
    print(form.errors.as_json())
```

### Adding Errors Programmatically

```python
def clean(self):
    cleaned_data = super().clean()
    
    # Add field-specific error
    self.add_error('email', 'This email is banned.')
    
    # Add non-field error
    self.add_error(None, 'Form submission blocked.')
    
    return cleaned_data
```

### Template Display

```html
{% if form.non_field_errors %}
<div class="alert alert-danger">
    {{ form.non_field_errors }}
</div>
{% endif %}

{% for field in form %}
    {{ field.label_tag }}
    {{ field }}
    {% if field.errors %}
    <ul class="errors">
        {% for error in field.errors %}{{ error }}{% endfor %}
    </ul>
    {% endif %}
{% endfor %}
```

## Summary

Access field errors via form.errors dict. Non-field errors come from clean(). Use add_error() for programmatic errors.

## Resources

- [Form Errors](https://docs.djangoproject.com/en/6.0/ref/forms/api/#django.forms.Form.errors) — Form errors documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*