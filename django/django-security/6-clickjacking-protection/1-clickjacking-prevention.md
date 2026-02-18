---
source_course: "django-security"
source_lesson: "django-security-clickjacking-prevention"
---

# Clickjacking Prevention

Clickjacking tricks users into clicking hidden elements by overlaying transparent iframes.

## How Clickjacking Works

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

## X-Frame-Options Header

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

## Per-View Control

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

## Content Security Policy (CSP)

```python
# Modern replacement for X-Frame-Options
# pip install django-csp

# settings.py
MIDDLEWARE = [
    # ...
    'csp.middleware.CSPMiddleware',
]

# Deny all framing
CSP_FRAME_ANCESTORS = ("'none'",)

# Allow same-origin only
CSP_FRAME_ANCESTORS = ("'self'",)

# Allow specific domains
CSP_FRAME_ANCESTORS = ("'self'", 'https://trusted-partner.com')
```

## Class-Based View Protection

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

## JavaScript Frame Busting (Fallback)

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

## Resources

- [Clickjacking Protection](https://docs.djangoproject.com/en/6.0/ref/clickjacking/) — Django clickjacking protection

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*