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

```html
<!-- templates/contact.html -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Send</button>
</form>
```

### Rendering Options

```html
{{ form.as_p }}      <!-- Wraps each field in <p> -->
{{ form.as_div }}    <!-- Wraps each field in <div> -->
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

## Resources

- [Working with Forms](https://docs.djangoproject.com/en/6.0/topics/forms/) — Official guide to Django forms

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*