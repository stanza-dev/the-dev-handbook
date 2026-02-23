---
source_course: "django-foundations"
source_lesson: "django-foundations-template-basics"
---

# Template Basics

Django templates are HTML files with special syntax for inserting dynamic content. The template system separates design from Python code.

## Setting Up Templates

Create a templates directory in your app:

```
polls/
├── templates/
│   └── polls/           # Namespaced to avoid conflicts
│       └── index.html
├── models.py
└── views.py
```

Or configure a project-wide templates directory in `settings.py`:

```python
# mysite/settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # Project-wide templates
        'APP_DIRS': True,  # Also check app templates/ directories
        ...
    },
]
```

## Basic Template Syntax

### Variables

Display values using double curly braces:

```html
<!-- polls/templates/polls/detail.html -->
<h1>{{ question.question_text }}</h1>
<p>Published: {{ question.pub_date }}</p>
```

Access attributes, methods, dictionary keys, and list items:

```html
{{ user.username }}          <!-- Attribute -->
{{ user.get_full_name }}      <!-- Method (no parentheses) -->
{{ my_dict.key }}             <!-- Dictionary key -->
{{ my_list.0 }}               <!-- List index -->
```

### Filters

Transform values with filters using the pipe `|` symbol:

```html
{{ name|lower }}              <!-- lowercase -->
{{ name|upper }}              <!-- UPPERCASE -->
{{ name|title }}              <!-- Title Case -->
{{ text|truncatewords:30 }}   <!-- Truncate to 30 words -->
{{ date|date:"F j, Y" }}      <!-- Format date -->
{{ value|default:"N/A" }}     <!-- Default if empty -->
{{ list|length }}             <!-- Count items -->
{{ price|floatformat:2 }}     <!-- Format number -->
{{ html|safe }}               <!-- Mark as safe HTML -->
```

### Tags

Tags provide logic and control flow:

```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
{% else %}
    <p>Please log in.</p>
{% endif %}

{% for question in questions %}
    <li>{{ question.question_text }}</li>
{% empty %}
    <li>No questions available.</li>
{% endfor %}
```

## Rendering Templates

### Using render()

The most common way to render templates:

```python
# polls/views.py
from django.shortcuts import render
from .models import Question


def index(request):
    questions = Question.objects.order_by('-pub_date')[:5]
    context = {
        'questions': questions,
        'page_title': 'Latest Questions',
    }
    return render(request, 'polls/index.html', context)
```

```html
<!-- polls/templates/polls/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ page_title }}</title>
</head>
<body>
    <h1>{{ page_title }}</h1>
    <ul>
    {% for question in questions %}
        <li>{{ question.question_text }}</li>
    {% endfor %}
    </ul>
</body>
</html>
```

### Template Context

The context is a dictionary mapping variable names to values:

```python
context = {
    'question': Question.objects.get(pk=1),
    'choices': Choice.objects.filter(question_id=1),
    'total_votes': 42,
    'show_results': True,
}
return render(request, 'polls/detail.html', context)
```

## Common Pitfalls

- **Calling methods with parentheses in templates**: In Django templates, write `{{ user.get_full_name }}` without parentheses. Adding `()` will cause a template syntax error.
- **Forgetting `{% load static %}` before using `{% static %}`**: The static tag requires loading the static template tag library first.
- **Not namespacing app templates**: Place templates in `app/templates/app/` (not just `app/templates/`) to avoid name collisions between apps.

## Best Practices

- **Keep templates simple**: Move complex logic to views or template tags, not inline template code.
- **Use template filters for formatting**: Apply `|date`, `|truncatewords`, and `|default` filters instead of formatting data in Python views.
- **Always escape user input**: Django auto-escapes variables by default. Only use `|safe` when you are certain the content is safe HTML.

## Summary

- Django templates combine HTML with `{{ variables }}` and `{% tags %}` for dynamic content
- Use `render(request, template, context)` to pass data from views to templates
- **Filters** (`|lower`, `|date`, `|default`) transform variable output
- **Tags** (`{% if %}`, `{% for %}`, `{% url %}`) provide logic and control flow
- Templates are stored in `app/templates/app/` directories for proper namespacing

## Code Examples

**Passing context data to a Django template using the render shortcut**

```python
from django.shortcuts import render
from .models import Question

def index(request):
    questions = Question.objects.order_by('-pub_date')[:5]
    context = {
        'questions': questions,
        'page_title': 'Latest Questions',
    }
    return render(request, 'polls/index.html', context)
```


## Resources

- [Django Templates](https://docs.djangoproject.com/en/6.0/topics/templates/) — Official guide to Django's template language

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*