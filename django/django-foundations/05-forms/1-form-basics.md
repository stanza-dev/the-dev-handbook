---
source_course: "django-foundations"
source_lesson: "django-foundations-form-basics"
---

# Introduction to Django Forms

Django's forms framework handles HTML form rendering, data validation, and security. It protects against common vulnerabilities like CSRF attacks.

## Why Use Django Forms?

- **Automatic HTML generation**: Create form fields with proper attributes
- **Validation**: Built-in and custom validation rules
- **Security**: CSRF protection included
- **Error handling**: Display validation errors easily
- **Data cleaning**: Normalize and sanitize input

## Creating a Form Class

Forms are defined as Python classes that inherit from `forms.Form`. Each class attribute represents a form field with its own validation rules.

```python
# polls/forms.py
from django import forms


class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
    
    # Optional fields
    phone = forms.CharField(max_length=20, required=False)
    subscribe = forms.BooleanField(required=False)
```

## Form Field Types

| Field Type | HTML Input | Description |
|------------|------------|-------------|
| `CharField` | `<input type="text">` | Single-line text |
| `EmailField` | `<input type="email">` | Email with validation |
| `IntegerField` | `<input type="number">` | Integer numbers |
| `FloatField` | `<input type="number">` | Decimal numbers |
| `BooleanField` | `<input type="checkbox">` | True/False |
| `ChoiceField` | `<select>` | Dropdown selection |
| `DateField` | `<input type="text">` | Date input |
| `FileField` | `<input type="file">` | File upload |
| `URLField` | `<input type="url">` | URL with validation |

## Using Forms in Views

In a view, you create a form instance for GET requests (empty form) and bind POST data to it for validation on submission.

```python
# polls/views.py
from django.shortcuts import render, redirect
from .forms import ContactForm


def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Process the data
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']
            # Send email, save to database, etc.
            return redirect('success')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})
```

## Rendering Forms in Templates

The simplest way to render a form is to use Django's built-in rendering methods inside a `<form>` tag with a CSRF token.

```html
<!-- templates/contact.html -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Send</button>
</form>
```

### Rendering Options

Django provides several methods that render the entire form with different HTML wrappers.

```html
{{ form.as_p }}      <!-- Wraps each field in <p> -->
{{ form.as_div }}    <!-- Wraps each field in <div> (default since Django 5.0) -->
{{ form.as_ul }}     <!-- Wraps each field in <li> -->
{{ form.as_table }}  <!-- Renders as <tr> (needs <table>) -->
```

### Manual Field Rendering

For full control over HTML:

```html
<form method="post">
    {% csrf_token %}
    
    <div class="field">
        <label for="{{ form.name.id_for_label }}">Name:</label>
        {{ form.name }}
        {% if form.name.errors %}
            <span class="error">{{ form.name.errors.0 }}</span>
        {% endif %}
    </div>
    
    <div class="field">
        <label for="{{ form.email.id_for_label }}">Email:</label>
        {{ form.email }}
        {{ form.email.errors }}
    </div>
    
    <button type="submit">Send</button>
</form>
```

## CSRF Protection

**Always include `{% csrf_token %}`** in your forms. Django uses this token to prevent Cross-Site Request Forgery attacks:

```html
<form method="post">
    {% csrf_token %}  <!-- Required for POST forms -->
    ...
</form>
```

Without this token, Django will reject the form submission with a 403 Forbidden error.

## Common Pitfalls

- **Forgetting `{% csrf_token %}`**: Without the CSRF token, Django rejects POST form submissions with a 403 Forbidden error.
- **Accessing `cleaned_data` before `is_valid()`**: You must call `form.is_valid()` first. Accessing `cleaned_data` on an unvalidated form raises an `AttributeError`.
- **Not re-rendering the form on validation errors**: When `is_valid()` returns `False`, render the same template with the form object to display error messages to the user.

## Best Practices

- **Always use POST for forms that modify data**: GET should only be used for search forms or read-only operations.
- **Use `form.as_div`** (Django 5.0+) or manual field rendering for production-quality HTML styling.
- **Redirect after successful POST**: Follow the Post/Redirect/Get pattern to prevent duplicate form submissions.

## Summary

- Django forms handle **rendering, validation, and CSRF protection** automatically
- Define form fields in a `forms.Form` subclass with field types like `CharField`, `EmailField`, `BooleanField`
- Check `request.method == 'POST'` and call `form.is_valid()` before accessing `cleaned_data`
- Always include `{% csrf_token %}` in form templates to prevent cross-site request forgery
- Render forms with `{{ form.as_p }}`, `{{ form.as_div }}`, or manual field-by-field rendering

## Code Examples

**Defining a Django form class and handling form submission with validation in a view**

```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)

# In a view
def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            name = form.cleaned_data['name']
            # Process data...
    else:
        form = ContactForm()
    return render(request, 'contact.html', {'form': form})
```


## Resources

- [Working with Forms](https://docs.djangoproject.com/en/6.0/topics/forms/) — Official guide to Django forms

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*