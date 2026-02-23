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

## Real World Context

In a user registration flow, you need to display field-specific errors inline next to the field, while showing cross-field errors at the top of the form. The `form.errors` dictionary and `form.non_field_errors()` method give you exactly this separation.

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

## Common Pitfalls

1. **Displaying `form.errors` without checking first** -- Always wrap in `{% if form.errors %}`.
2. **Calling `add_error()` outside of `clean()`** -- Only call during validation.
3. **Forgetting that `add_error()` removes the field from `cleaned_data`** -- Subsequent code must handle the missing key.

## Best Practices

1. **Use `form.errors.as_json()` for API responses** -- Provides structured error data easy to parse.
2. **Show non-field errors prominently** -- Place at the top of the form in a visible alert box.

## Summary

Access field errors via form.errors dict. Non-field errors come from clean(). Use add_error() for programmatic errors.

## Resources

- [Form Errors](https://docs.djangoproject.com/en/6.0/ref/forms/api/#django.forms.Form.errors) — Form errors documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*