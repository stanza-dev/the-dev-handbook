---
source_course: "django-security"
source_lesson: "django-security-csrf-fundamentals"
---

# CSRF Protection

Cross-Site Request Forgery (CSRF) tricks users into performing unwanted actions on a site where they're authenticated.

## How CSRF Works

```html
<!-- Malicious site -->
<form action="https://yourbank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="10000">
    <input type="hidden" name="to" value="attacker-account">
    <input type="submit" value="Click for free iPhone!">
</form>
```

## Django's CSRF Protection

Django includes automatic CSRF protection via middleware:

```python
# settings.py - Enabled by default
MIDDLEWARE = [
    # ...
    'django.middleware.csrf.CsrfViewMiddleware',
    # ...
]
```

## Using CSRF in Templates

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

## CSRF in AJAX Requests

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

## CSRF Settings

```python
# settings.py

# Cookie settings
CSRF_COOKIE_SECURE = True        # Only send over HTTPS
CSRF_COOKIE_HTTPONLY = True      # Not accessible via JavaScript (use header instead)
CSRF_COOKIE_SAMESITE = 'Strict'  # Don't send with cross-origin requests
CSRF_COOKIE_NAME = 'csrftoken'   # Cookie name

# Trusted origins for HTTPS
CSRF_TRUSTED_ORIGINS = [
    'https://example.com',
    'https://www.example.com',
]

# Use a custom failure view
CSRF_FAILURE_VIEW = 'myapp.views.csrf_failure'
```

## Exempting Views from CSRF

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

## CSRF in REST APIs

```python
# For session-authenticated APIs, CSRF is still needed
# For token-based APIs (JWT, OAuth), CSRF is not needed

from rest_framework.authentication import TokenAuthentication

class MyAPIView(APIView):
    authentication_classes = [TokenAuthentication]
    # No CSRF needed - token auth doesn't use cookies
```

## Resources

- [CSRF Protection](https://docs.djangoproject.com/en/6.0/howto/csrf/) — Django CSRF protection guide

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*