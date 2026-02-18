---
source_course: "django-foundations"
source_lesson: "django-foundations-template-tags-filters"
---

# Built-in Tags and Filters

Django provides many built-in template tags and filters for common tasks. Let's explore the most useful ones.

## Control Flow Tags

### if / elif / else

```html
{% if user.is_authenticated %}
    <p>Welcome back, {{ user.username }}!</p>
{% elif user.is_anonymous %}
    <p>Welcome, guest!</p>
{% else %}
    <p>Hello!</p>
{% endif %}

<!-- Comparison operators -->
{% if count > 10 %}
{% if count >= 10 %}
{% if count < 10 %}
{% if count == 10 %}
{% if count != 10 %}

<!-- Boolean operators -->
{% if user.is_authenticated and user.is_staff %}
{% if user.is_superuser or user.is_staff %}
{% if not user.is_authenticated %}

<!-- In operator -->
{% if 'python' in languages %}
{% if user not in blocked_users %}
```

### for loop

```html
{% for item in items %}
    <p>{{ item }}</p>
{% empty %}
    <p>No items found.</p>
{% endfor %}

<!-- Loop variables -->
{% for item in items %}
    {{ forloop.counter }}     <!-- 1, 2, 3, ... -->
    {{ forloop.counter0 }}    <!-- 0, 1, 2, ... -->
    {{ forloop.first }}       <!-- True for first iteration -->
    {{ forloop.last }}        <!-- True for last iteration -->
    {{ forloop.revcounter }}  <!-- Counts down to 1 -->
{% endfor %}

<!-- Iterate over dictionary -->
{% for key, value in data.items %}
    {{ key }}: {{ value }}
{% endfor %}
```

## URL Tag

Generate URLs dynamically:

```html
<a href="{% url 'polls:index' %}">All Polls</a>
<a href="{% url 'polls:detail' question.id %}">Details</a>
<a href="{% url 'polls:detail' pk=question.id %}">Details</a>

<!-- With query string -->
<a href="{% url 'search' %}?q={{ query }}">Search</a>
```

## Static Files Tag

Reference static files:

```html
{% load static %}

<link rel="stylesheet" href="{% static 'css/style.css' %}">
<script src="{% static 'js/app.js' %}"></script>
<img src="{% static 'images/logo.png' %}" alt="Logo">
```

## Common Filters

### String Filters

```html
{{ name|lower }}              <!-- hello world -->
{{ name|upper }}              <!-- HELLO WORLD -->
{{ name|title }}              <!-- Hello World -->
{{ name|capfirst }}           <!-- Hello world -->
{{ text|truncatewords:10 }}   <!-- First 10 words... -->
{{ text|truncatechars:50 }}   <!-- First 50 chars... -->
{{ text|wordcount }}          <!-- 5 -->
{{ text|linebreaks }}         <!-- Converts \n to <br> and <p> -->
{{ text|linebreaksbr }}       <!-- Converts \n to <br> -->
{{ text|striptags }}          <!-- Remove HTML tags -->
{{ slug|slugify }}            <!-- hello-world -->
```

### Number Filters

```html
{{ price|floatformat }}       <!-- 34.2 -->
{{ price|floatformat:2 }}     <!-- 34.20 -->
{{ count|add:5 }}             <!-- Add 5 to count -->
{{ number|divisibleby:2 }}    <!-- True/False -->
```

### Date Filters

```html
{{ date|date:"F j, Y" }}      <!-- January 5, 2024 -->
{{ date|date:"D d M Y" }}     <!-- Fri 05 Jan 2024 -->
{{ date|date:"Y-m-d" }}       <!-- 2024-01-05 -->
{{ date|time:"H:i" }}         <!-- 14:30 -->
{{ date|timesince }}          <!-- 2 days ago -->
{{ date|timeuntil }}          <!-- in 3 days -->
```

### List Filters

```html
{{ items|length }}            <!-- Count items -->
{{ items|first }}             <!-- First item -->
{{ items|last }}              <!-- Last item -->
{{ items|join:", " }}         <!-- a, b, c -->
{{ items|slice:":3" }}        <!-- First 3 items -->
{{ items|random }}            <!-- Random item -->
```

### Default Values

```html
{{ value|default:"N/A" }}            <!-- N/A if falsy -->
{{ value|default_if_none:"N/A" }}    <!-- N/A only if None -->
{{ items|yesno:"Yes,No,Maybe" }}     <!-- Based on True/False/None -->
```

### Safety Filters

```html
{{ html_content|safe }}       <!-- Don't escape HTML -->
{{ user_input|escape }}       <!-- Escape HTML (default) -->
{{ content|escapejs }}        <!-- Safe for JavaScript -->
```

## Resources

- [Built-in Template Tags and Filters](https://docs.djangoproject.com/en/6.0/ref/templates/builtins/) — Complete reference of built-in template tags and filters

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*