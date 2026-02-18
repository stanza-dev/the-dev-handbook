---
source_course: "django-security"
source_lesson: "django-security-html-sanitization"
---

# HTML Sanitization

## Introduction

Sometimes you need to allow users to submit HTML (rich text editors, markdown). Sanitization removes dangerous elements while preserving safe formatting.

## Key Concepts

**Sanitization**: Removing or escaping dangerous HTML elements and attributes.

**Allowlist**: Only permit explicitly safe tags and attributes.

**Bleach**: Popular Python library for HTML sanitization.

## Deep Dive

### Using Bleach

```python
# pip install bleach
import bleach

ALLOWED_TAGS = ['p', 'br', 'strong', 'em', 'a', 'ul', 'ol', 'li', 'code', 'pre']
ALLOWED_ATTRS = {
    'a': ['href', 'title'],
    '*': ['class'],
}

def sanitize_html(html):
    return bleach.clean(
        html,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRS,
        strip=True
    )

# Usage in model
class Article(models.Model):
    content_raw = models.TextField()
    content_safe = models.TextField(editable=False)
    
    def save(self, *args, **kwargs):
        self.content_safe = sanitize_html(self.content_raw)
        super().save(*args, **kwargs)
```

### Link Sanitization

```python
import bleach

def sanitize_with_safe_links(html):
    return bleach.clean(
        html,
        tags=['a', 'p', 'br'],
        attributes={'a': ['href']},
        protocols=['http', 'https', 'mailto'],  # Block javascript:
        strip=True
    )
```

### Markdown to Safe HTML

```python
import markdown
import bleach

def markdown_to_safe_html(text):
    # Convert markdown to HTML
    html = markdown.markdown(text)
    # Sanitize the result
    return sanitize_html(html)
```

## Best Practices

1. **Sanitize on save, not display**: Process once, serve many times.
2. **Keep allowlists minimal**: Only allow tags you really need.
3. **Block javascript: URLs**: Use protocol allowlists.

## Summary

When allowing user HTML, sanitize with an allowlist approach using Bleach. Process on save, keep allowlists minimal, and always block dangerous URL protocols.

## Resources

- [Bleach Documentation](https://bleach.readthedocs.io/) — Bleach HTML sanitization library

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*