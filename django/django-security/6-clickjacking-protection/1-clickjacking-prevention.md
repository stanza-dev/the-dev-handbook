---
source_course: "django-security"
source_lesson: "django-security-clickjacking-prevention"
---

# Clickjacking Prevention

## Introduction

Clickjacking is a UI redress attack where an attacker overlays a transparent iframe on top of a legitimate page, tricking users into clicking hidden elements. Django's XFrameOptionsMiddleware provides automatic protection by controlling whether your pages can be embedded in frames.

## Key Concepts

- **Clickjacking**: Attack that overlays invisible iframes to hijack user clicks.
- **X-Frame-Options**: HTTP header that controls whether browsers allow your page to be framed.
- **frame-ancestors**: CSP directive that supersedes X-Frame-Options with more flexible domain control.
- **xframe_options_exempt**: Django decorator to selectively allow framing for specific views.

## Real World Context

Clickjacking has been used to trick users into enabling webcams, liking Facebook pages (likejacking), changing privacy settings, and transferring money. Any authenticated action that requires a single click is vulnerable if the page can be framed. Financial institutions and social platforms are prime targets.

## Deep Dive

### How Clickjacking Works

```html
<!-- Attacker's page -->
<style>
    iframe {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        opacity: 0.01;  /* Nearly invisible */
        z-index: 1;
    }
    button {
        position: absolute;
        top: 100px;
        left: 100px;
        z-index: 0;
    }
</style>

<button>Click for Prize!</button>
<iframe src="https://yourbank.com/transfer?amount=1000"></iframe>
<!-- User clicks the button but actually clicks in the iframe -->
```

### X-Frame-Options Header

```python
# settings.py

# Enabled by default via XFrameOptionsMiddleware
MIDDLEWARE = [
    # ...
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# Options:
X_FRAME_OPTIONS = 'DENY'        # Never allow framing
X_FRAME_OPTIONS = 'SAMEORIGIN'  # Only allow same-origin framing
```

### Per-View Control

```python
from django.views.decorators.clickjacking import (
    xframe_options_deny,
    xframe_options_sameorigin,
    xframe_options_exempt
)


@xframe_options_deny
def sensitive_view(request):
    """Never allow this view to be framed."""
    return render(request, 'sensitive.html')


@xframe_options_sameorigin
def embeddable_widget(request):
    """Allow framing only from same origin."""
    return render(request, 'widget.html')


@xframe_options_exempt
def public_embed(request):
    """Allow any site to frame this (use carefully!)."""
    return render(request, 'public_embed.html')
```

### Content Security Policy (CSP)

Django 6.0+ includes built-in CSP support as a modern replacement for X-Frame-Options:

```python
from django.utils.csp import CSP

# settings.py
MIDDLEWARE = [
    # ...
    'django.middleware.csp.ContentSecurityPolicyMiddleware',
]

# Deny all framing
SECURE_CSP = {
    'frame-ancestors': [CSP.NONE],
}

# Allow same-origin only
SECURE_CSP = {
    'frame-ancestors': [CSP.SELF],
}

# Allow specific domains
SECURE_CSP = {
    'frame-ancestors': [CSP.SELF, 'https://trusted-partner.com'],
}
```

### Class-Based View Protection

```python
from django.utils.decorators import method_decorator
from django.views.decorators.clickjacking import xframe_options_deny


@method_decorator(xframe_options_deny, name='dispatch')
class SecureView(View):
    def get(self, request):
        return render(request, 'secure.html')


# Or use a mixin
class XFrameDenyMixin:
    @method_decorator(xframe_options_deny)
    def dispatch(self, *args, **kwargs):
        return super().dispatch(*args, **kwargs)


class MySecureView(XFrameDenyMixin, TemplateView):
    template_name = 'secure.html'
```

### JavaScript Frame Busting (Fallback)

```html
<!-- Last resort - can be bypassed -->
<script>
    if (self !== top) {
        top.location = self.location;
    }
</script>

<!-- Better: Use sandbox attribute -->
<style>
    html { display: none; }
</style>
<script>
    if (self === top) {
        document.documentElement.style.display = 'block';
    } else {
        top.location = self.location;
    }
</script>
```

## Common Pitfalls

1. **Removing XFrameOptionsMiddleware for embedded widgets** — Instead of disabling global protection, use @xframe_options_exempt on specific views that need framing.
2. **Using SAMEORIGIN when DENY suffices** — Default to DENY; only use SAMEORIGIN if you actively embed your own pages in iframes.
3. **Relying on JavaScript frame busting alone** — Attackers can use sandbox attributes on iframes to block JavaScript execution.

## Best Practices

1. **Set X_FRAME_OPTIONS = 'DENY' globally** — Override per-view only where embedding is explicitly needed.
2. **Use CSP frame-ancestors alongside X-Frame-Options** — CSP is more flexible and supported by modern browsers; X-Frame-Options covers older browsers.
3. **Require confirmation for sensitive actions** — Even with framing protection, add re-authentication or CAPTCHA for high-value operations.

## Summary

- Django's XFrameOptionsMiddleware adds X-Frame-Options headers to all responses by default.
- Set X_FRAME_OPTIONS = 'DENY' globally and use per-view decorators for exceptions.
- Combine X-Frame-Options with CSP frame-ancestors for comprehensive framing protection across all browsers.

## Code Examples

**Global X-Frame-Options with DENY default and per-view decorators for selective framing exceptions**

```python
# settings.py - Global clickjacking protection
MIDDLEWARE = [
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # ...
]
X_FRAME_OPTIONS = 'DENY'

# views.py - Per-view exceptions
from django.views.decorators.clickjacking import (
    xframe_options_exempt,
    xframe_options_sameorigin,
)

@xframe_options_exempt
def public_widget(request):
    """This view can be embedded anywhere."""
    return render(request, 'widget.html')

@xframe_options_sameorigin
def internal_embed(request):
    """Only allow framing from same origin."""
    return render(request, 'embed.html')
```


## Resources

- [Clickjacking Protection](https://docs.djangoproject.com/en/6.0/ref/clickjacking/) — Django clickjacking protection

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*