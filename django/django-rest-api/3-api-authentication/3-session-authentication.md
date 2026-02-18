---
source_course: "django-rest-api"
source_lesson: "django-rest-api-session-authentication"
---

# Session Authentication for APIs

## Introduction

While tokens are popular for APIs, session authentication remains valuable—especially for web applications where your frontend and backend share the same domain. Django's built-in session framework provides battle-tested security with minimal configuration.

## Key Concepts

**Session Authentication**: Using Django's session cookies to identify users. The session ID is stored in a cookie and maps to user data on the server.

**CSRF Protection**: Required for session-based APIs since browsers automatically send cookies. The CSRF token prevents cross-site request forgery.

**Same-Origin Policy**: Session auth works best when frontend and backend share the same origin (domain + port).

## Real World Context

Session authentication is ideal for:
- **Traditional web apps**: Django templates with AJAX calls
- **Same-domain SPAs**: React/Vue apps served from the same domain
- **Admin interfaces**: Internal tools where simplicity matters
- **Hybrid applications**: Mix of server-rendered pages and API calls

## Deep Dive

### Enabling Session Authentication

Django sessions are enabled by default. Verify your settings:

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.sessions',
    'django.contrib.auth',
    # ...
]

MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    # ...
]
```

### Login/Logout API Endpoints

```python
import json
from django.http import JsonResponse
from django.contrib.auth import authenticate, login, logout
from django.views.decorators.http import require_POST
from django.middleware.csrf import get_token

def api_csrf_token(request):
    """Return CSRF token for AJAX requests."""
    return JsonResponse({'csrfToken': get_token(request)})

@require_POST
def api_login(request):
    data = json.loads(request.body)
    user = authenticate(
        request,
        username=data.get('username'),
        password=data.get('password')
    )
    
    if user is None:
        return JsonResponse({'error': 'Invalid credentials'}, status=401)
    
    login(request, user)
    return JsonResponse({
        'user': {'id': user.id, 'username': user.username}
    })

@require_POST
def api_logout(request):
    logout(request)
    return JsonResponse({'message': 'Logged out'})

def api_current_user(request):
    if request.user.is_authenticated:
        return JsonResponse({
            'user': {'id': request.user.id, 'username': request.user.username}
        })
    return JsonResponse({'user': None})
```

### CSRF with AJAX

```javascript
// Get CSRF token from cookie
function getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
}

// Include in requests
fetch('/api/articles/', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': getCookie('csrftoken'),
    },
    credentials: 'same-origin',  // Important!
    body: JSON.stringify({title: 'New Article'})
});
```

### Secure Session Settings

```python
# settings.py (production)
SESSION_COOKIE_SECURE = True        # HTTPS only
SESSION_COOKIE_HTTPONLY = True      # No JavaScript access
SESSION_COOKIE_SAMESITE = 'Lax'     # CSRF protection
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_HTTPONLY = False        # JS needs to read this
```

## Common Pitfalls

1. **Forgetting credentials: 'same-origin'**: Fetch doesn't send cookies by default. Always include credentials option.

2. **CORS issues**: Session auth doesn't work well cross-origin. Use tokens for cross-domain APIs.

3. **Missing CSRF token**: POST/PUT/DELETE requests need the X-CSRFToken header.

## Best Practices

1. **Use sessions for same-origin**: When frontend and backend share a domain, sessions are simpler than tokens.

2. **Secure your cookies**: Always set Secure, HttpOnly, and SameSite flags in production.

3. **Provide a CSRF endpoint**: Let SPAs fetch a fresh CSRF token on page load.

## Summary

Session authentication leverages Django's built-in session framework for API security. It works best for same-origin applications where cookies are automatically sent. Remember to handle CSRF tokens for non-GET requests and configure secure cookie settings in production.

## Resources

- [Session Framework](https://docs.djangoproject.com/en/6.0/topics/http/sessions/) — Django sessions documentation
- [CSRF Protection](https://docs.djangoproject.com/en/6.0/howto/csrf/) — CSRF protection with AJAX

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*