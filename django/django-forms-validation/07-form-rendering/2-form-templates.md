---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-custom-form-templates"
---

# Custom Form Templates

## Introduction

Since Django 4.0, form rendering is driven by templates rather than Python string generation. You can customize how every form on your site renders by overriding these templates, or set per-form templates for fine-grained control.

## Key Concepts

- **form_template_name**: The template used when rendering the entire form (e.g., {{ form }}).
- **field_template_name**: The template used for each individual field.
- **FORM_RENDERER**: A setting that controls which renderer class Django uses for all forms.

## Real World Context

Every design system has its own markup pattern for form fields (Bootstrap wraps fields in specific div classes, Tailwind uses utility classes). Custom form templates let you define this markup once and have every form on your site render consistently without per-field widget attrs.

## Deep Dive

### Setting a Global Form Template

Configure a custom renderer in settings.py to change how all forms render:

```python
# settings.py
from django.forms.renderers import TemplatesSetting


class CustomFormRenderer(TemplatesSetting):
    form_template_name = "forms/custom_form.html"
    field_template_name = "forms/custom_field.html"


FORM_RENDERER = "myproject.settings.CustomFormRenderer"
```

Now every {{ form }} call in your templates uses your custom markup.

### Creating the Form Template

The form template receives the form object in context:

```html
{# templates/forms/custom_form.html #}
{% for field in form.hidden_fields %}
    {{ field }}
{% endfor %}

{% for field in form.visible_fields %}
    <div class="mb-4">
        <label for="{{ field.id_for_label }}" class="block font-medium mb-1">
            {{ field.label }}
            {% if field.field.required %}
                <span class="text-red-500">*</span>
            {% endif %}
        </label>
        {{ field }}
        {% if field.help_text %}
            <p class="text-sm text-gray-500 mt-1">{{ field.help_text }}</p>
        {% endif %}
        {% for error in field.errors %}
            <p class="text-sm text-red-600 mt-1">{{ error }}</p>
        {% endfor %}
    </div>
{% endfor %}
```

This template applies Tailwind-style classes to every form on the site.

### Per-Form Template Override

Override the template for a specific form class:

```python
class ContactForm(forms.Form):
    template_name = "forms/contact_form.html"
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
```

Or render with a specific template in the view:

```python
def contact_view(request):
    form = ContactForm()
    rendered = form.render("forms/inline_form.html")
    return render(request, "contact.html", {"form": rendered})
```

### Using as_field_group()

Django 5.0 introduced as_field_group() for rendering individual fields with their associated label, help text, and errors:

```html
<form method="post">
    {% csrf_token %}
    {{ form.name.as_field_group }}
    {{ form.email.as_field_group }}
    {{ form.message.as_field_group }}
    <button type="submit">Send</button>
</form>
```

Each field renders using the field_template_name, giving you consistent per-field markup.

## Common Pitfalls

1. **Template not found errors** — When using TemplatesSetting as your renderer base, your custom templates must be in a directory listed in TEMPLATES[0]['DIRS']. The default DjangoTemplates renderer only looks in app template dirs.
2. **Forgetting hidden fields** — Custom templates must render hidden fields separately or they get lost, breaking CSRF and other hidden inputs.

## Best Practices

1. **Start with Django's built-in templates** — Copy django/forms/templates/django/forms/div.html as your starting point, then modify. This ensures you don't miss edge cases.
2. **Use field_template_name for consistency** — A shared field template ensures every field across every form renders identically, reducing CSS maintenance.

## Summary

- Set FORM_RENDERER with a custom renderer class to control site-wide form rendering.
- Override form_template_name and field_template_name for global templates.
- Use template_name on individual form classes for per-form overrides.
- Use as_field_group() for rendering individual fields with their full markup.

## Resources

- [Reusable Form Templates](https://docs.djangoproject.com/en/6.0/topics/forms/#reusable-form-templates) — Official guide to template-based form rendering

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*