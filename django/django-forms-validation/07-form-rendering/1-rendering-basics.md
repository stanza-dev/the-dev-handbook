---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-rendering-basics"
---

# Form Rendering Options

Django provides multiple ways to render forms, from automatic to completely manual control.

## Quick Rendering Methods

```html
<!-- As paragraph elements -->
{{ form.as_p }}

<!-- As table rows (needs <table> wrapper) -->
<table>{{ form.as_table }}</table>

<!-- As unordered list -->
<ul>{{ form.as_ul }}</ul>

<!-- As div elements (Django 4.0+) -->
{{ form.as_div }}
```

## Rendering Individual Fields

```html
<form method="post">
    {% csrf_token %}
    
    <div class="form-group">
        {{ form.title.label_tag }}
        {{ form.title }}
        {{ form.title.help_text }}
        {% if form.title.errors %}
            <div class="errors">{{ form.title.errors }}</div>
        {% endif %}
    </div>
    
    <div class="form-group">
        {{ form.body.label_tag }}
        {{ form.body }}
    </div>
    
    <button type="submit">Submit</button>
</form>
```

## Field Attributes

```html
<!-- Access various field properties -->
{{ form.title.name }}           <!-- 'title' -->
{{ form.title.id_for_label }}   <!-- 'id_title' -->
{{ form.title.value }}          <!-- Current value -->
{{ form.title.html_name }}      <!-- HTML name attribute -->
{{ form.title.help_text }}      <!-- Help text -->
{{ form.title.label }}          <!-- Label text -->
{{ form.title.errors }}         <!-- Field errors -->
{{ form.title.is_hidden }}      <!-- Is hidden field -->
```

## Looping Over Fields

```html
<form method="post">
    {% csrf_token %}
    
    {% for field in form %}
        <div class="form-group {% if field.errors %}has-error{% endif %}">
            {{ field.label_tag }}
            {{ field }}
            
            {% if field.help_text %}
                <small class="help-text">{{ field.help_text }}</small>
            {% endif %}
            
            {% for error in field.errors %}
                <span class="error">{{ error }}</span>
            {% endfor %}
        </div>
    {% endfor %}
    
    <button type="submit">Submit</button>
</form>
```

## Hidden Fields

```html
<form method="post">
    {% csrf_token %}
    
    <!-- Render hidden fields first -->
    {% for hidden in form.hidden_fields %}
        {{ hidden }}
    {% endfor %}
    
    <!-- Render visible fields -->
    {% for field in form.visible_fields %}
        <div class="form-group">
            {{ field.label_tag }}
            {{ field }}
        </div>
    {% endfor %}
</form>
```

## Custom Widget Attributes

```python
# forms.py
class ContactForm(forms.Form):
    name = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Your name',
            'autocomplete': 'name',
        })
    )
    
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'your@email.com',
        })
    )
    
    message = forms.CharField(
        widget=forms.Textarea(attrs={
            'class': 'form-control',
            'rows': 5,
        })
    )
```

## Non-Field Errors

```html
<!-- Display form-level errors -->
{% if form.non_field_errors %}
    <div class="alert alert-danger">
        {% for error in form.non_field_errors %}
            <p>{{ error }}</p>
        {% endfor %}
    </div>
{% endif %}

<!-- All errors summary -->
{% if form.errors %}
    <div class="alert alert-danger">
        <strong>Please correct the errors below:</strong>
        {{ form.errors }}
    </div>
{% endif %}
```

## Form Templates (Django 4.0+)

```python
# settings.py
FORM_RENDERER = 'django.forms.renderers.TemplatesSetting'

TEMPLATES = [{
    # ...
    'DIRS': [BASE_DIR / 'templates'],
}]
```

```html
<!-- templates/django/forms/div.html -->
{% for field in form %}
<div class="form-group mb-3">
    {% if field.label %}
        <label for="{{ field.id_for_label }}" class="form-label">
            {{ field.label }}
            {% if field.field.required %}<span class="required">*</span>{% endif %}
        </label>
    {% endif %}
    
    {{ field }}
    
    {% if field.help_text %}
        <div class="form-text">{{ field.help_text }}</div>
    {% endif %}
    
    {% for error in field.errors %}
        <div class="invalid-feedback d-block">{{ error }}</div>
    {% endfor %}
</div>
{% endfor %}
```

## Resources

- [Working with Form Templates](https://docs.djangoproject.com/en/6.0/topics/forms/#working-with-form-templates) — Form rendering documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*