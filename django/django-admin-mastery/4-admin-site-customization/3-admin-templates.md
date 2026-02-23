---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-templates"
---

# Customizing Admin Templates

## Introduction

Override Django admin templates to customize the look and feel of your admin interface.

## Key Concepts

**Template override**: Place templates in your project to override defaults.

**change_list_template**: Custom template for the list view.

**change_form_template**: Custom template for the edit form.

## Real World Context

A company needs its admin to match its brand guidelines with a custom logo, color scheme, and a 'Quick Links' sidebar. By extending base_site.html and overriding the branding block, plus adding a custom CSS file via the extrastyle block, the admin looks professional without replacing any Django functionality.

## Deep Dive

### Template Override Hierarchy

```
templates/
└── admin/
    ├── base_site.html          # Override site-wide base
    ├── index.html               # Override admin index
    └── myapp/
        └── mymodel/
            ├── change_list.html  # Override list view
            └── change_form.html  # Override edit form
```

### Override Base Site Template

```html
<!-- templates/admin/base_site.html -->
{% extends "admin/base.html" %}
{% load static %}

{% block title %}{{ title }} | My Company{% endblock %}

{% block branding %}
<div id="site-name">
    <a href="{% url 'admin:index' %}">
        <img src="{% static 'img/logo.png' %}" alt="Logo" height="40">
        My Company Admin
    </a>
</div>
{% endblock %}

{% block extrastyle %}
<link rel="stylesheet" href="{% static 'admin/css/custom.css' %}">
{% endblock %}
```

### Custom Change List Template

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    change_list_template = 'admin/blog/article/change_list.html'
```

```html
<!-- templates/admin/blog/article/change_list.html -->
{% extends "admin/change_list.html" %}

{% block object-tools-items %}
<li><a href="export/" class="button">Export CSV</a></li>
{{ block.super }}
{% endblock %}
```

## Common Pitfalls

1. **Replacing templates instead of extending them**: Creating a template from scratch (without `{% extends %}`) means you lose all default admin functionality including CSRF tokens, JavaScript, and message display. Always extend the corresponding default template.

2. **Not using block.super**: Omitting `{{ block.super }}` in an overridden block removes the default content entirely. Use it to add to the existing content rather than replacing it.

## Best Practices

1. **Always extend**: Never replace admin templates from scratch.
2. **Use block.super**: Preserve default functionality.

## Summary

Override admin templates by creating matching paths. Always extend base templates. Use per-app and per-model overrides for targeted changes.

## Resources

- [Admin Template Override](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#overriding-admin-templates) — Overriding admin templates

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*