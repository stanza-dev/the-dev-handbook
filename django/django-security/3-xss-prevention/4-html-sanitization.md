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

**nh3**: Recommended Rust-based HTML sanitizer (bleach was deprecated in 2023).



## Real World Context

Rich text editors (TinyMCE, CKEditor) produce HTML that must be sanitized server-side before storage. Client-side sanitization is easily bypassed by sending raw POST requests. The bleach library served this role for years but was deprecated in 2023 in favor of the faster, Rust-based nh3.

## Deep Dive

### Using nh3

```python
# pip install nh3  (nh3 is the recommended HTML sanitizer; bleach was deprecated in 2023)
import nh3

ALLOWED_TAGS = {'p', 'br', 'strong', 'em', 'a', 'ul', 'ol', 'li', 'code', 'pre'}
ALLOWED_ATTRS = {
    'a': {'href', 'title'},
}

def sanitize_html(html):
    return nh3.clean(
        html,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRS,
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
import nh3

def sanitize_with_safe_links(html):
    return nh3.clean(
        html,
        tags={'a', 'p', 'br'},
        attributes={'a': {'href'}},
        url_schemes={'http', 'https', 'mailto'},  # Block javascript:
    )
```

### Markdown to Safe HTML

```python
import markdown
import nh3

def markdown_to_safe_html(text):
    # Convert markdown to HTML
    html = markdown.markdown(text)
    # Sanitize the result
    return sanitize_html(html)
```



## Common Pitfalls

1. **Sanitizing on display instead of on save** — Sanitizing on every page load wastes CPU and risks inconsistency; sanitize once when saving to the database.
2. **Using an overly permissive allowlist** — Allowing style attributes or event handler attributes (onclick, onerror) re-opens XSS vectors even in sanitized HTML.

## Best Practices

1. **Sanitize on save, not display**: Process once, serve many times.
2. **Keep allowlists minimal**: Only allow tags you really need.
3. **Block javascript: URLs**: Use protocol allowlists.

## Summary

When allowing user HTML, sanitize with an allowlist approach using nh3. Process on save, keep allowlists minimal, and always block dangerous URL protocols.

## Resources

- [nh3 Documentation](https://nh3.readthedocs.io/) — nh3 HTML sanitization library (Rust-based successor to bleach)

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*