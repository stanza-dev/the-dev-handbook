---
source_course: "django-security"
source_lesson: "django-security-csp-framing"
---

# CSP Frame-Ancestors

## Introduction

Content Security Policy's frame-ancestors directive is the modern replacement for X-Frame-Options. It offers more flexibility, including allowing specific domains to frame your content.

## Key Concepts

**frame-ancestors**: CSP directive controlling who can embed your page.

**More Flexible**: Unlike X-Frame-Options, allows specific domains.

**Supersedes X-Frame-Options**: Browsers prefer CSP when both are set.

## Deep Dive

### Using django-csp

```python
# pip install django-csp

# settings.py
MIDDLEWARE = [
    'csp.middleware.CSPMiddleware',
    ...
]

# No framing allowed (like DENY)
CSP_FRAME_ANCESTORS = ("'none'",)

# Same origin only (like SAMEORIGIN)
CSP_FRAME_ANCESTORS = ("'self'",)

# Specific trusted domains
CSP_FRAME_ANCESTORS = (
    "'self'",
    'https://trusted-partner.com',
    'https://app.example.com',
)
```

### Per-View CSP

```python
from csp.decorators import csp_update

@csp_update(FRAME_ANCESTORS="'self' https://partner.com")
def partnered_widget(request):
    pass
```

### Migration from X-Frame-Options

```python
# Keep both during transition
X_FRAME_OPTIONS = 'DENY'
CSP_FRAME_ANCESTORS = ("'none'",)

# Modern browsers use CSP, older use X-Frame-Options
```

## Best Practices

1. **Use CSP for flexibility**: When you need specific domain allowlists.
2. **Keep X-Frame-Options**: For older browser support.
3. **Use 'none' by default**: More secure than 'self'.

## Summary

frame-ancestors provides fine-grained control over framing. Use django-csp to set it globally, and per-view decorators for exceptions. Keep X-Frame-Options alongside CSP for maximum browser compatibility.

## Resources

- [CSP frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) — MDN frame-ancestors documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*