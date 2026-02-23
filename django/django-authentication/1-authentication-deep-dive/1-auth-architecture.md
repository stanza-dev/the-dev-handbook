---
source_course: "django-authentication"
source_lesson: "django-authentication-auth-architecture"
---

# Django Authentication Architecture

## Introduction

Django's authentication system is a comprehensive framework that handles user accounts, groups, permissions, and cookie-based sessions. Understanding its architecture is essential for building secure web applications.

## Key Concepts

**Authentication**: Verifying a user's identity (login).

**Authorization**: Determining what an authenticated user can do (permissions).

**Middleware**: Request processing pipeline that enables auth.

**Backends**: Pluggable authentication verification strategies.

## Real World Context

Every web application needs authentication. Django provides a battle-tested system used by sites like Instagram, Disqus, and Mozilla. Instead of building auth from scratch, Django gives you a secure foundation to build on.

## Deep Dive

### Authentication Pipeline

```
Request → SecurityMiddleware → SessionMiddleware → AuthenticationMiddleware → View
```

1. **SessionMiddleware** attaches a `session` attribute to the request
2. **AuthenticationMiddleware** attaches a `user` attribute using the session
3. **Views** can then check `request.user` for the current user

### Core Components

```python
# The auth app provides these models
from django.contrib.auth.models import User, Group, Permission

# And these key functions
from django.contrib.auth import (
    authenticate,  # Verify credentials
    login,         # Create session
    logout,        # Destroy session
)
```

### Authentication Flow

```python
from django.contrib.auth import authenticate, login, logout
from django.shortcuts import redirect, render

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']

        # authenticate() checks credentials against backends
        user = authenticate(request, username=username, password=password)

        if user is not None:
            # login() creates session and attaches user
            login(request, user)
            return redirect('dashboard')
        else:
            return render(request, 'login.html', {
                'error': 'Invalid credentials'
            })

    return render(request, 'login.html')

def logout_view(request):
    logout(request)  # Clears session
    return redirect('home')
```

### Settings Configuration

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    ...
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]

# Authentication backends (checked in order)
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
]
```

## Common Pitfalls

1. **Middleware order matters**: `SessionMiddleware` must come before `AuthenticationMiddleware`.
2. **Forgetting `contenttypes`**: The auth app depends on `django.contrib.contenttypes`.
3. **Not using `authenticate()`**: Always use `authenticate()` instead of manual password checks.

## Best Practices

1. **Use `@login_required`** decorator for protected views.
2. **Set `AUTH_USER_MODEL`** at the start of a project if you need a custom user.
3. **Never store plaintext passwords**: Django handles hashing automatically.

## Summary

Django's authentication architecture is a middleware-driven pipeline. `SessionMiddleware` manages sessions, `AuthenticationMiddleware` attaches users. The `authenticate()` function checks credentials against configured backends, and `login()` creates sessions.

## Code Examples

**Basic authentication flow using Django's authenticate() and login() functions**

```python
from django.contrib.auth import authenticate, login

def login_view(request):
    username = request.POST['username']
    password = request.POST['password']
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
        return redirect('dashboard')
    return render(request, 'login.html', {'error': 'Invalid credentials'})
```

**Django authentication middleware and backend configuration in settings.py**

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
]
```


## Resources

- [User Authentication in Django](https://docs.djangoproject.com/en/6.0/topics/auth/) — Official authentication documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*