---
source_course: "django-security"
source_lesson: "django-security-frame-options-configuration"
---

# X-Frame-Options Configuration

## Introduction

X-Frame-Options is a simple but effective HTTP header that controls whether your page can be embedded in frames. It's your first line of defense against clickjacking.

## Key Concepts

**DENY**: Never allow framing, by anyone, ever.

**SAMEORIGIN**: Only allow framing from the same origin (domain + protocol + port).

**ALLOW-FROM**: Deprecated—use CSP frame-ancestors instead.

## Deep Dive

### Global Configuration

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ...
]

# DENY - most secure, use for most pages
X_FRAME_OPTIONS = 'DENY'

# SAMEORIGIN - if you need to frame your own pages
X_FRAME_OPTIONS = 'SAMEORIGIN'
```

### Per-View Configuration

```python
from django.views.decorators.clickjacking import (
    xframe_options_deny,
    xframe_options_sameorigin,
    xframe_options_exempt
)

# Most restrictive for sensitive pages
@xframe_options_deny
def account_settings(request):
    pass

# Allow embedding your own widgets
@xframe_options_sameorigin
def embed_widget(request):
    pass

# Public content that can be embedded anywhere
@xframe_options_exempt
def public_video(request):
    pass
```

## Best Practices

1. **Default to DENY**: Unless you specifically need framing.
2. **Use SAMEORIGIN sparingly**: Only for internal embedding needs.
3. **Document exemptions**: Comment why each exempt view is safe.

## Summary

Set X_FRAME_OPTIONS = 'DENY' globally and selectively allow framing for specific views that require it. DENY is the safest default for protecting against clickjacking.

## Resources

- [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) — MDN X-Frame-Options documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*