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



## Real World Context

GitHub uses strict CSP to protect its users. When they first deployed CSP, they used report-only mode for months to identify legitimate scripts that would break. Skipping this step and enforcing immediately often breaks third-party analytics, chat widgets, and payment forms.

## Deep Dive

### Django 6.0+ Native CSP (Recommended)

Django 6.0 includes built-in CSP support with no third-party package needed:

```python
# settings.py
from django.utils.csp import CSP

MIDDLEWARE = [
    # ...
    'django.middleware.csp.ContentSecurityPolicyMiddleware',
    # ...
]

SECURE_CSP = {
    'default-src': [CSP.SELF],
    'script-src': [CSP.SELF, CSP.NONCE],
    'style-src': [CSP.SELF, CSP.UNSAFE_INLINE],
    'img-src': [CSP.SELF, 'data:', 'https:'],
    'font-src': [CSP.SELF, 'https://fonts.gstatic.com'],
    'connect-src': [CSP.SELF, 'https://api.example.com'],
}

# Report-only mode for testing
SECURE_CSP_REPORT_ONLY = {
    'default-src': [CSP.SELF],
    'report-uri': ['/csp-report/'],
}
```

### Nonce Support (Django 6.0+)

```python
# In templates settings, add the CSP context processor
TEMPLATES = [{
    'OPTIONS': {
        'context_processors': [
            'django.template.context_processors.csp',
        ],
    },
}]
```

```html
<script nonce="{{ csp_nonce }}">
    // This inline script is allowed
</script>
```

### Legacy: Using django-csp (Django < 6.0)

For Django < 6.0, use the third-party django-csp package:

```python
# pip install django-csp

MIDDLEWARE = ['csp.middleware.CSPMiddleware', ...]

# settings.py
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'", 'https://cdn.example.com')
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
CSP_IMG_SRC = ("'self'", 'data:', 'https:')
CSP_FONT_SRC = ("'self'", 'https://fonts.gstatic.com')
CSP_CONNECT_SRC = ("'self'", 'https://api.example.com')

CSP_INCLUDE_NONCE_IN = ['script-src']
CSP_REPORT_ONLY = True
CSP_REPORT_URI = '/csp-report/'
```



## Common Pitfalls

1. **Enforcing CSP without report-only testing** — Strict policies can break third-party scripts, payment forms, and analytics; always test in report-only mode first.
2. **Using unsafe-inline instead of nonces** — unsafe-inline defeats much of CSP's XSS protection; migrate to nonces for inline scripts.

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