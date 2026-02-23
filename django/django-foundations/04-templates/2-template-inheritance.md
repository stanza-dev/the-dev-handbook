---
source_course: "django-foundations"
source_lesson: "django-foundations-template-inheritance"
---

# Template Inheritance

Template inheritance lets you build a base "skeleton" template with common elements and define blocks that child templates can override. This is one of Django's most powerful features.

## Base Template

Create a base template with blocks:

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Site{% endblock %}</title>
    {% block extra_css %}{% endblock %}
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'home' %}">Home</a>
            <a href="{% url 'polls:index' %}">Polls</a>
        </nav>
    </header>

    <main>
        {% block content %}
        <!-- Child templates fill this in -->
        {% endblock %}
    </main>

    <footer>
        <p>&copy; 2024 My Site</p>
    </footer>

    {% block extra_js %}{% endblock %}
</body>
</html>
```

## Child Template

Extend the base and override blocks:

```html
<!-- polls/templates/polls/index.html -->
{% extends 'base.html' %}

{% block title %}Polls - My Site{% endblock %}

{% block content %}
<h1>Latest Questions</h1>
<ul>
{% for question in questions %}
    <li>
        <a href="{% url 'polls:detail' question.id %}">
            {{ question.question_text }}
        </a>
    </li>
{% empty %}
    <li>No questions available.</li>
{% endfor %}
</ul>
{% endblock %}
```

## Key Rules

1. `{% extends %}` must be the first tag in the template
2. More blocks = more flexibility
3. Child templates only override blocks they need
4. Use `{{ block.super }}` to include parent content

## Using block.super

Include the parent block's content:

```html
<!-- base.html -->
{% block sidebar %}
<nav>
    <a href="/">Home</a>
    <a href="/about/">About</a>
</nav>
{% endblock %}

<!-- child.html -->
{% extends 'base.html' %}

{% block sidebar %}
{{ block.super }}  <!-- Include parent nav -->
<nav>
    <a href="/polls/">Polls</a>  <!-- Add more links -->
</nav>
{% endblock %}
```

## Three-Level Inheritance

A common pattern for complex sites:

```
base.html              (Site-wide structure)
    └── base_polls.html    (Section-specific)
            └── index.html     (Page-specific)
```

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>

<!-- polls/templates/polls/base_polls.html -->
{% extends 'base.html' %}

{% block body %}
<div class="polls-layout">
    <aside>{% block sidebar %}{% endblock %}</aside>
    <main>{% block content %}{% endblock %}</main>
</div>
{% endblock %}

<!-- polls/templates/polls/index.html -->
{% extends 'polls/base_polls.html' %}

{% block sidebar %}
<h3>Categories</h3>
<ul>...</ul>
{% endblock %}

{% block content %}
<h1>Polls</h1>
...
{% endblock %}
```

## Include Tag

For reusable snippets that don't need inheritance:

```html
<!-- templates/includes/navbar.html -->
<nav>
    <a href="/">Home</a>
    <a href="/about/">About</a>
</nav>

<!-- base.html -->
<header>
    {% include 'includes/navbar.html' %}
</header>

<!-- Pass variables to includes -->
{% include 'includes/card.html' with title=question.question_text %}
```

## Common Pitfalls

- **Not putting `{% extends %}` as the first tag**: The `{% extends %}` tag must be the very first template tag in a child template, or Django will not recognize the inheritance.
- **Forgetting `{% endblock %}`**: Every `{% block name %}` must have a matching `{% endblock %}`. Missing it causes a template syntax error.
- **Overusing `{% include %}` instead of inheritance**: Use inheritance for page structure and `{% include %}` only for reusable snippets like navigation or cards.

## Best Practices

- **Define many blocks in the base template**: More blocks give child templates more flexibility to override specific sections.
- **Use `{{ block.super }}`** when you want to extend (not replace) a parent block's content.
- **Follow three-level inheritance** for complex sites: `base.html` -> `base_section.html` -> `page.html`.

## Summary

- Template inheritance lets you define a **base skeleton** with `{% block %}` placeholders
- Child templates use `{% extends 'base.html' %}` and override specific blocks
- `{% extends %}` must be the **first tag** in a child template
- Use `{{ block.super }}` to include the parent block's content alongside new content
- `{% include %}` is for reusable snippets; inheritance is for page structure

## Code Examples

**Template inheritance with a base template defining blocks and a child template overriding them**

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <main>{% block content %}{% endblock %}</main>
</body>
</html>

<!-- child.html -->
{% extends "base.html" %}
{% block title %}My Page{% endblock %}
{% block content %}<h1>Hello!</h1>{% endblock %}
```


## Resources

- [Template Inheritance](https://docs.djangoproject.com/en/6.0/ref/templates/language/#template-inheritance) — Official documentation on template inheritance

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*