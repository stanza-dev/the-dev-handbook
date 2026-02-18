---
source_course: "django-authentication"
source_lesson: "django-authentication-authentication-backends"
---

# Custom Authentication Backends

Authentication backends control how users are authenticated. Create custom backends for email login, LDAP, OAuth, or any custom logic.

## Backend Interface

A backend must implement:

```python
class MyBackend:
    def authenticate(self, request, **credentials):
        """Return a user if credentials are valid, else None."""
        pass
    
    def get_user(self, user_id):
        """Return user by ID, used to restore user from session."""
        pass
```

## Email Authentication Backend

```python
# backends.py
from django.contrib.auth import get_user_model
from django.contrib.auth.backends import ModelBackend

User = get_user_model()


class EmailBackend(ModelBackend):
    """
    Authenticate using email address instead of username.
    """
    
    def authenticate(self, request, username=None, password=None, **kwargs):
        # 'username' parameter actually contains the email
        email = kwargs.get('email', username)
        
        if email is None or password is None:
            return None
        
        try:
            user = User.objects.get(email__iexact=email)
        except User.DoesNotExist:
            # Run password hasher to prevent timing attacks
            User().set_password(password)
            return None
        
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
        
        return None
```

```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'myapp.backends.EmailBackend',
    'django.contrib.auth.backends.ModelBackend',  # Fallback to username
]
```

## Email OR Username Backend

```python
class EmailOrUsernameBackend(ModelBackend):
    """
    Allow authentication with either email or username.
    """
    
    def authenticate(self, request, username=None, password=None, **kwargs):
        if username is None or password is None:
            return None
        
        User = get_user_model()
        
        # Try to find user by email first, then username
        try:
            if '@' in username:
                user = User.objects.get(email__iexact=username)
            else:
                user = User.objects.get(username__iexact=username)
        except User.DoesNotExist:
            User().set_password(password)
            return None
        
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
        
        return None
```

## Case-Insensitive Username Backend

```python
class CaseInsensitiveBackend(ModelBackend):
    """
    Case-insensitive username authentication.
    """
    
    def authenticate(self, request, username=None, password=None, **kwargs):
        if username is None:
            return None
        
        User = get_user_model()
        
        try:
            user = User.objects.get(username__iexact=username)
        except User.DoesNotExist:
            User().set_password(password)
            return None
        except User.MultipleObjectsReturned:
            # Handle edge case of duplicate usernames (shouldn't happen)
            user = User.objects.filter(username__iexact=username).first()
        
        if user and user.check_password(password) and self.user_can_authenticate(user):
            return user
        
        return None
```

## Token-Based Backend

```python
from .models import AuthToken


class TokenBackend:
    """
    Authenticate via bearer token.
    """
    
    def authenticate(self, request, token=None, **kwargs):
        if token is None:
            return None
        
        try:
            auth_token = AuthToken.objects.select_related('user').get(
                key=token,
                is_active=True
            )
            
            if auth_token.is_expired:
                return None
            
            return auth_token.user
        except AuthToken.DoesNotExist:
            return None
    
    def get_user(self, user_id):
        User = get_user_model()
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
```

## Multiple Backend Strategy

```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'myapp.backends.TokenBackend',           # Try token first (API)
    'myapp.backends.EmailOrUsernameBackend', # Then email/username
    'django.contrib.auth.backends.ModelBackend', # Fallback
]
```

Django tries backends in order until one returns a user or all return `None`.

## Resources

- [Customizing Authentication](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/) — Official guide to customizing authentication

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*