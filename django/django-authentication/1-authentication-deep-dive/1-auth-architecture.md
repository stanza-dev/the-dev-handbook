---
source_course: "django-authentication"
source_lesson: "django-authentication-auth-architecture"
---

# Authentication Architecture

Django's authentication system is composed of several interconnected components. Understanding this architecture helps you customize and extend it effectively.

## Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                        Request                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              AuthenticationMiddleware                        │
│         (Sets request.user from session)                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Authentication Backends                       │
│    (ModelBackend, RemoteUserBackend, Custom...)              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────┐      │     ┌─────────────────────────┐
│      User Model      │◄─────┼────►│    Permission Model     │
│   (AbstractUser)     │      │     │  (User/Group Perms)     │
└──────────────────────┘      │     └─────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Session Backend                           │
│         (Database, Cache, File, Cookie)                      │
└─────────────────────────────────────────────────────────────┘
```

## The User Model

Django's default User model provides:

```python
from django.contrib.auth.models import User

# User fields
user.username        # Required, unique
user.password        # Hashed password
user.email           # Email address
user.first_name      # Optional
user.last_name       # Optional
user.is_active       # Can this user log in?
user.is_staff        # Can access admin?
user.is_superuser    # Has all permissions?
user.date_joined     # When created
user.last_login      # Last login time

# Important methods
user.set_password('newpassword')     # Hash and set
user.check_password('password')      # Verify password
user.has_perm('app.permission')      # Check permission
user.get_all_permissions()           # All permissions
```

## Authentication Backends

Backends determine how credentials are validated:

```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',  # Default
]
```

The `authenticate()` function tries each backend:

```python
from django.contrib.auth import authenticate

# Tries each backend in order
user = authenticate(request, username='john', password='secret')

if user is not None:
    # Credentials valid
    pass
else:
    # Invalid credentials
    pass
```

## Session Management

Django stores authentication state in sessions:

```python
from django.contrib.auth import login, logout

# After successful authentication
login(request, user)  # Creates session, sets cookie

# Session now contains:
# - _auth_user_id: User's primary key
# - _auth_user_backend: Backend that authenticated
# - _auth_user_hash: Password hash (invalidates on password change)

# To log out
logout(request)  # Clears session
```

## The Anonymous User

```python
from django.contrib.auth.models import AnonymousUser

# When not authenticated, request.user is AnonymousUser
if request.user.is_authenticated:
    # Real user
    print(request.user.username)
else:
    # AnonymousUser
    print(request.user)  # AnonymousUser
```

`AnonymousUser` has the same interface but:
- `is_authenticated` returns `False`
- `is_anonymous` returns `True`
- Has no permissions
- `id` is always `None`

## Middleware Order Matters

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',  # Required first
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',  # After session
    'django.contrib.messages.middleware.MessageMiddleware',
]
```

`SessionMiddleware` must come before `AuthenticationMiddleware` because auth depends on sessions.

## Resources

- [User Authentication in Django](https://docs.djangoproject.com/en/6.0/topics/auth/) — Official authentication documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*