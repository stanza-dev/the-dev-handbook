---
source_course: "django-security"
source_lesson: "django-security-csrf-spa"
---

# CSRF with Single Page Applications

## Introduction

SPAs (React, Vue, Angular) require special handling for CSRF tokens since they don't use traditional form submissions.

## Key Concepts

**SPA CSRF Flow**: Get token from cookie, include in request header.

**credentials: 'include'**: Fetch option to send cookies with requests.

**CORS Configuration**: Required for cross-origin SPA to backend communication.

## Deep Dive

### Setting Up SPA CSRF

```python
# Django settings
CSRF_COOKIE_HTTPONLY = False  # Allow JS to read token
CSRF_COOKIE_SAMESITE = 'Lax'
CSRF_TRUSTED_ORIGINS = ['https://spa.example.com']

# CORS settings (django-cors-headers)
CORS_ALLOWED_ORIGINS = ['https://spa.example.com']
CORS_ALLOW_CREDENTIALS = True
```

### JavaScript Implementation

```javascript
// Get CSRF token from cookie
function getCSRFToken() {
    const name = 'csrftoken';
    const cookies = document.cookie.split(';');
    for (let cookie of cookies) {
        cookie = cookie.trim();
        if (cookie.startsWith(name + '=')) {
            return cookie.substring(name.length + 1);
        }
    }
    return null;
}

// Configure Axios/Fetch
const api = axios.create({
    baseURL: 'https://api.example.com',
    withCredentials: true,  // Send cookies
    headers: {
        'X-CSRFToken': getCSRFToken(),
    }
});

// Or with Fetch
fetch('/api/data/', {
    method: 'POST',
    credentials: 'include',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': getCSRFToken(),
    },
    body: JSON.stringify(data)
});
```

### Endpoint to Get Fresh Token

```python
from django.middleware.csrf import get_token
from django.http import JsonResponse

def get_csrf_token(request):
    return JsonResponse({'csrfToken': get_token(request)})
```

## Best Practices

1. **First request should get token**: Call CSRF endpoint on app init.
2. **Include credentials**: Always use credentials: 'include' for cookies.
3. **Handle token expiry**: Refresh token if request fails with 403.

## Summary

SPAs need CSRF_COOKIE_HTTPONLY=False to read the token, must include credentials in requests, and should fetch a fresh token on application init.

## Resources

- [CSRF and AJAX](https://docs.djangoproject.com/en/6.0/howto/csrf/#using-csrf-protection-with-ajax) — CSRF with AJAX requests

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*