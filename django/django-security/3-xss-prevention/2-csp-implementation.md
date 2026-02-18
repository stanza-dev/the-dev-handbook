---
source_course: "django-security"
source_lesson: "django-security-csp-implementation"
---

# Content Security Policy

## Introduction

Content Security Policy (CSP) is a browser security feature that helps prevent XSS attacks by controlling which resources can be loaded and executed on your page.

## Key Concepts

**CSP Header**: HTTP header that defines allowed sources for scripts, styles, images, etc.

**Directives**: Rules like script-src, style-src that control specific resource types.

**Nonces**: One-time tokens for inline scripts that need to be allowed.

## Deep Dive

### Basic CSP Configuration

```python
# Using django-csp
# pip install django-csp

MIDDLEWARE = ['csp.middleware.CSPMiddleware', ...]

# settings.py
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'", 'https://cdn.example.com')
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
CSP_IMG_SRC = ("'self'", 'data:', 'https:')
CSP_FONT_SRC = ("'self'", 'https://fonts.gstatic.com')
CSP_CONNECT_SRC = ("'self'", 'https://api.example.com')
```

### Using Nonces for Inline Scripts

```python
# settings.py
CSP_INCLUDE_NONCE_IN = ['script-src']
```

```html
<script nonce="{{ request.csp_nonce }}">
    // This inline script is allowed
</script>
```

### Report-Only Mode

```python
# Test CSP without blocking
CSP_REPORT_ONLY = True
CSP_REPORT_URI = '/csp-report/'
```

## Best Practices

1. **Start with report-only**: Test before enforcing.
2. **Avoid unsafe-inline**: Use nonces instead.
3. **Be specific**: List exact domains, not wildcards.

## Summary

CSP provides defense-in-depth against XSS. Configure strict policies, use nonces for necessary inline scripts, and test with report-only mode before enforcing.

## Resources

- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) — MDN CSP documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*