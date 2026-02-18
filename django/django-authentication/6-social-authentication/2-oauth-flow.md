---
source_course: "django-authentication"
source_lesson: "django-authentication-oauth-flow"
---

# OAuth 2.0 Flow

## Introduction

Understand the OAuth 2.0 authorization flow that powers social login.

## Key Concepts

**Authorization Code Flow**: Most secure for web apps.

**Access Token**: Grants API access.

## Deep Dive

### OAuth 2.0 Flow

```
1. User clicks 'Login with Google'
2. Redirect to provider's authorization endpoint
3. User grants permission
4. Provider redirects back with authorization code
5. Exchange code for access token
6. Fetch user info with access token
7. Create/login local user
```

### django-allauth Handles This

```python
# urls.py
urlpatterns = [
    path('accounts/', include('allauth.urls')),
]

# Endpoints created:
# /accounts/google/login/
# /accounts/google/login/callback/
```

### Callback URL

```python
# Configure in Google Console:
# http://localhost:8000/accounts/google/login/callback/
# https://yourdomain.com/accounts/google/login/callback/
```

## Best Practices

1. **Use HTTPS**: Always in production.
2. **Validate state parameter**: Prevents CSRF.
3. **Minimal scopes**: Request only needed data.

## Summary

OAuth 2.0 securely delegates authentication. django-allauth handles the complex flow. Always use HTTPS and minimal scopes.

## Resources

- [OAuth 2.0](https://oauth.net/2/) — OAuth 2.0 specification

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*