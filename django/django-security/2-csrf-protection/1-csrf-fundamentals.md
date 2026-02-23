---
source_course: "django-security"
source_lesson: "django-security-csrf-fundamentals"
---

# CSRF Protection

## Introduction

Cross-Site Request Forgery (CSRF) is an attack that tricks authenticated users into performing unwanted actions on your site. Django's CSRF middleware is enabled by default and protects every POST, PUT, PATCH, and DELETE request automatically.

## Key Concepts

- **CSRF Attack**: A malicious site submits a form to your domain using the victim's browser cookies, executing actions the user never intended.
- **CSRF Token**: A unique, unpredictable value that Django embeds in forms and validates on submission.
- **CsrfViewMiddleware**: Django middleware that enforces CSRF token validation on unsafe HTTP methods.
- **CSRF_TRUSTED_ORIGINS**: Setting that whitelists domains for cross-origin HTTPS requests.

## Real World Context

CSRF attacks have been used to transfer funds from bank accounts, change email addresses on social media, and modify admin settings on CMS platforms. Because the browser automatically attaches session cookies, the server cannot distinguish a legitimate form submission from a forged one without a CSRF token.

## Deep Dive

### How CSRF Works

```html
<!-- Malicious site -->
<form action="https://yourbank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="10000">
    <input type="hidden" name="to" value="attacker-account">
    <input type="submit" value="Click for free iPhone!">
</form>
```

### Django's CSRF Protection

Django includes automatic CSRF protection via middleware:

```python
# settings.py - Enabled by default
MIDDLEWARE = [
    # ...
    'django.middleware.csrf.CsrfViewMiddleware',
    # ...
]
```

### Using CSRF in Templates

```html
<!-- All POST forms must include the token -->
<form method="post">
    {% csrf_token %}
    <input type="text" name="title">
    <button type="submit">Submit</button>
</form>

<!-- The token is rendered as -->
<input type="hidden" name="csrfmiddlewaretoken" value="abc123...">
```

### CSRF in AJAX Requests

```javascript
// Get CSRF token from cookie
function getCookie(name) {
    let cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            const cookie = cookies[i].trim();
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

const csrftoken = getCookie('csrftoken');

// Include in AJAX requests
fetch('/api/submit/', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': csrftoken,
    },
    body: JSON.stringify({data: 'value'}),
});
```

### CSRF Settings

```python
# settings.py

# Cookie settings
CSRF_COOKIE_SECURE = True        # Only send over HTTPS
CSRF_COOKIE_HTTPONLY = False       # Default. Set True only if not reading token via JS
# Note: If True, JavaScript cannot read the CSRF cookie.
# You must then get the token from a hidden form input instead.
CSRF_COOKIE_SAMESITE = 'Lax'     # Send on navigation, not on cross-origin POST
CSRF_COOKIE_NAME = 'csrftoken'   # Cookie name

# Trusted origins for HTTPS
CSRF_TRUSTED_ORIGINS = [
    'https://example.com',
    'https://www.example.com',
]

# Use a custom failure view
CSRF_FAILURE_VIEW = 'myapp.views.csrf_failure'
```

### Exempting Views from CSRF

```python
from django.views.decorators.csrf import csrf_exempt, csrf_protect


@csrf_exempt
def webhook_endpoint(request):
    """External webhooks don't have CSRF tokens."""
    # Verify webhook signature instead!
    if not verify_webhook_signature(request):
        return HttpResponseForbidden()
    
    # Process webhook
    return HttpResponse('OK')


# Re-enable CSRF for specific methods in a class-based view
from django.utils.decorators import method_decorator


@method_decorator(csrf_exempt, name='dispatch')
class WebhookView(View):
    def post(self, request):
        # Handle webhook
        pass
```

### CSRF in REST APIs

```python
# For session-authenticated APIs, CSRF is still needed
# For token-based APIs (JWT, OAuth), CSRF is not needed

from rest_framework.authentication import TokenAuthentication

class MyAPIView(APIView):
    authentication_classes = [TokenAuthentication]
    # No CSRF needed - token auth doesn't use cookies
```

## Common Pitfalls

1. **Forgetting {% csrf_token %} in forms** — Results in 403 Forbidden errors that are hard to debug in production.
2. **Using @csrf_exempt on session-authenticated views** — Completely disables protection, making the view vulnerable to forged requests.
3. **Not configuring CSRF_TRUSTED_ORIGINS for HTTPS** — Django 4.0+ requires this for cross-origin POST requests over HTTPS.

## Best Practices

1. **Never remove CsrfViewMiddleware** — It protects all unsafe methods globally.
2. **Use X-CSRFToken header for AJAX** — Read the token from the cookie and send it as a header.
3. **Verify webhooks with signatures** — Use HMAC or provider-specific verification instead of @csrf_exempt alone.

## Summary

- CSRF protection is built into Django and works automatically for form submissions.
- AJAX requests must include the CSRF token via the X-CSRFToken header.
- Only exempt views from CSRF when using non-cookie authentication or webhook signature verification.

## Code Examples

**Two ways to include CSRF tokens: template tag for forms and X-CSRFToken header for AJAX requests**

```python
# Template: Include CSRF token in every POST form
# <form method="post">
#     {% csrf_token %}
#     <button type="submit">Submit</button>
# </form>

# AJAX: Send CSRF token via header
import requests

def get_csrf_cookie(cookie_jar):
    return cookie_jar.get('csrftoken')

# fetch('/api/data/', {
#     method: 'POST',
#     headers: { 'X-CSRFToken': getCookie('csrftoken') },
#     credentials: 'include',
#     body: JSON.stringify(data)
# });
```


## Resources

- [CSRF Protection](https://docs.djangoproject.com/en/6.0/howto/csrf/) — Django CSRF protection guide

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*