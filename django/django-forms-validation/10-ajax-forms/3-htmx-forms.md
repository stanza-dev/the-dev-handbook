---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-htmx-forms"
---

# HTMX with Django Forms

## Introduction

HTMX is a lightweight JavaScript library that lets you make AJAX requests directly from HTML attributes, without writing JavaScript. Combined with Django's template system, it enables partial page updates for form submission and validation with minimal code.

## Key Concepts

- **hx-post**: An HTML attribute that submits a form via AJAX to a URL.
- **hx-target**: Specifies which DOM element receives the response HTML.
- **hx-swap**: Controls how the response replaces content (innerHTML, outerHTML, etc.).
- **Partial templates**: Django templates that render just a fragment of the page, not the full HTML document.

## Real World Context

HTMX is popular in the Django community because it leverages Django's template engine for the response instead of requiring a separate JavaScript frontend. You return HTML fragments from your views, and HTMX swaps them into the page. This keeps your stack simple: Python + HTML.

## Deep Dive

### Basic HTMX Form Submission

Include HTMX from a CDN and add attributes to your form:

```html
{# base.html #}
<script src="https://unpkg.com/htmx.org@2.0.0"></script>

{# contact.html #}
<div id="form-container">
    <form hx-post="{% url 'contact_submit' %}"
          hx-target="#form-container"
          hx-swap="outerHTML">
        {% csrf_token %}
        {{ form.as_div }}
        <button type="submit">Send Message</button>
    </form>
</div>
```

When submitted, HTMX sends a POST request and replaces #form-container with the response HTML.

### Django View for HTMX

The view returns a full page for normal requests or a partial template for HTMX:

```python
from django.shortcuts import render, redirect


def contact_submit(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            form.save()
            # Return success fragment
            return render(request, 'partials/contact_success.html')
        else:
            # Return the form with errors
            return render(request, 'partials/contact_form.html', {
                'form': form,
            })

    form = ContactForm()
    template = 'partials/contact_form.html'
    if not request.headers.get('HX-Request'):
        template = 'contact.html'
    return render(request, template, {'form': form})
```

The HX-Request header tells you whether the request came from HTMX or a regular browser navigation.

### Partial Templates

Partial templates render just the form, not the full page layout:

```html
{# partials/contact_form.html #}
<div id="form-container">
    <form hx-post="{% url 'contact_submit' %}"
          hx-target="#form-container"
          hx-swap="outerHTML">
        {% csrf_token %}
        {{ form.as_div }}
        <button type="submit">Send Message</button>
    </form>
</div>

{# partials/contact_success.html #}
<div id="form-container">
    <div class="bg-green-100 p-4 rounded">
        <p>Thank you! Your message has been sent.</p>
    </div>
</div>
```

The success template replaces the form entirely with a confirmation message.

### Inline Field Validation with HTMX

Validate individual fields on blur without JavaScript:

```html
<input type="text" name="username"
       hx-get="{% url 'validate_username' %}"
       hx-trigger="blur changed delay:400ms"
       hx-target="#username-errors"
       hx-include="[name='username']">
<div id="username-errors"></div>
```

The view returns a small HTML fragment with error messages:

```python
def validate_username(request):
    username = request.GET.get('username', '')
    errors = []
    if len(username) < 3:
        errors.append('Must be at least 3 characters')
    if User.objects.filter(username=username).exists():
        errors.append('Already taken')

    return render(request, 'partials/field_errors.html', {
        'errors': errors,
        'valid': len(errors) == 0,
    })
```

```html
{# partials/field_errors.html #}
{% if errors %}
    {% for error in errors %}
        <p class="text-red-600 text-sm">{{ error }}</p>
    {% endfor %}
{% elif valid %}
    <p class="text-green-600 text-sm">Looks good!</p>
{% endif %}
```

The hx-trigger="blur changed delay:400ms" means: validate when the field loses focus, only if the value changed, with a 400ms debounce.

## Common Pitfalls

1. **CSRF token missing** — HTMX POST requests need the CSRF token. Include {% csrf_token %} inside the form or configure HTMX to send it as a header.
2. **Full page returned instead of partial** — If your view returns the full base template, HTMX will swap the entire page structure into the target div, breaking the layout. Always return partial HTML for HTMX requests.

## Best Practices

1. **Check HX-Request header** — Use request.headers.get('HX-Request') to detect HTMX requests and return the appropriate template (partial vs full page).
2. **Keep partials small** — Partial templates should render just the component being updated. This keeps responses fast and DOM updates minimal.

## Summary

- HTMX submits forms via AJAX using HTML attributes, no JavaScript required.
- Django views return HTML fragments that HTMX swaps into the page.
- Use partial templates for form fragments and full templates for initial page loads.
- hx-trigger with blur, changed, and delay provides inline field validation.

## Resources

- [HTMX Documentation](https://htmx.org/docs/) — Official HTMX documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*