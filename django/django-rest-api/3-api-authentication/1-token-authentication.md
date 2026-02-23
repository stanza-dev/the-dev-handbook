---
source_course: "django-rest-api"
source_lesson: "django-rest-api-token-authentication"
---

# Token-Based Authentication

## Introduction

Token authentication is the standard for API security. Instead of sending credentials with every request, clients receive a token after login and include it in subsequent requests. This approach is stateless, scalable, and works across different domains and platforms.

Token authentication is the standard for API security. Instead of sending credentials with every request, clients receive a token after login.

## Key Concepts

**API Token**: A unique, random string that identifies a user. Stored in the database and sent via the Authorization header.

**Bearer Authentication**: The standard format for sending tokens: `Authorization: Bearer <token>`.

**Token Lifecycle**: Tokens are created on login, sent with every request, and can expire or be revoked.

**Middleware**: A Django component that intercepts requests to extract and validate tokens before they reach your views.

## Real World Context

Token authentication is used in:
- **Mobile applications**: Tokens stored securely on device
- **Third-party API access**: Developers get API keys for their applications
- **Microservices**: Services authenticate with each other using tokens
- **CLI tools**: Command-line tools that interact with APIs

## How Token Auth Works

```
1. Client sends username/password to login endpoint
2. Server validates credentials and returns a token
3. Client stores the token
4. Client sends token in Authorization header with every request
5. Server validates token and identifies the user
```

## Creating a Token Model

The token model stores a cryptographically random key linked to a user, with optional expiration and usage tracking:

```python
# models.py
import secrets
from django.db import models
from django.conf import settings


class APIToken(models.Model):
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='api_tokens'
    )
    key = models.CharField(max_length=64, unique=True, db_index=True)
    name = models.CharField(max_length=100, blank=True)  # e.g., "Mobile App"
    created_at = models.DateTimeField(auto_now_add=True)
    last_used_at = models.DateTimeField(null=True, blank=True)
    expires_at = models.DateTimeField(null=True, blank=True)
    
    def save(self, *args, **kwargs):
        if not self.key:
            self.key = secrets.token_hex(32)  # 64 character hex string
        super().save(*args, **kwargs)
    
    def __str__(self):
        return f"{self.user.username} - {self.name or 'API Token'}"
    
    @property
    def is_expired(self):
        if self.expires_at is None:
            return False
        from django.utils import timezone
        return timezone.now() > self.expires_at
```

The `save()` override auto-generates a 64-character hex token using `secrets.token_hex(32)` when no key is set. The `is_expired` property checks against the current time.

## Login Endpoint

This view accepts credentials via POST, validates them with Django's `authenticate()`, and returns a token the client stores for future requests:

```python
# views.py
import json
from django.http import JsonResponse
from django.contrib.auth import authenticate
from django.views.decorators.csrf import csrf_exempt
from .models import APIToken


@csrf_exempt
def api_login(request):
    if request.method != 'POST':
        return JsonResponse({'error': 'Method not allowed'}, status=405)
    
    try:
        data = json.loads(request.body)
    except json.JSONDecodeError:
        return JsonResponse({'error': 'Invalid JSON'}, status=400)
    
    username = data.get('username')
    password = data.get('password')
    
    if not username or not password:
        return JsonResponse({'error': 'Username and password required'}, status=400)
    
    user = authenticate(username=username, password=password)
    
    if user is None:
        return JsonResponse({'error': 'Invalid credentials'}, status=401)
    
    if not user.is_active:
        return JsonResponse({'error': 'Account disabled'}, status=401)
    
    # Create or get token
    token, created = APIToken.objects.get_or_create(
        user=user,
        name='default'
    )
    
    return JsonResponse({
        'token': token.key,
        'user': {
            'id': user.id,
            'username': user.username,
            'email': user.email
        }
    })
```

The `get_or_create` call ensures each user gets one default token. For multiple tokens per user (mobile, desktop), pass a unique `name` for each.

## Token Authentication Middleware

This middleware intercepts every request, extracts the Bearer token from the Authorization header, and attaches the corresponding user to `request.user`:

```python
# middleware.py
from django.utils import timezone
from .models import APIToken


class TokenAuthMiddleware:
    """Authenticate requests using Bearer tokens."""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Get Authorization header
        auth_header = request.headers.get('Authorization', '')
        
        if auth_header.startswith('Bearer '):
            token_key = auth_header[7:]  # Remove 'Bearer ' prefix
            
            try:
                token = APIToken.objects.select_related('user').get(key=token_key)
                
                if not token.is_expired:
                    request.user = token.user
                    request.auth_token = token
                    
                    # Update last used
                    token.last_used_at = timezone.now()
                    token.save(update_fields=['last_used_at'])
            except APIToken.DoesNotExist:
                pass  # Invalid token, user remains anonymous
        
        return self.get_response(request)
```

Add the middleware to your Django settings so it runs on every request:

```python
# settings.py
MIDDLEWARE = [
    # ... other middleware
    'myapp.middleware.TokenAuthMiddleware',
]
```

The middleware uses `select_related('user')` to fetch the user in the same query as the token lookup, and tracks `last_used_at` for security auditing.

## Authentication Decorator

With the middleware setting `request.user`, you can protect individual views with a simple decorator that checks authentication:

```python
# decorators.py
from functools import wraps
from django.http import JsonResponse


def api_login_required(view_func):
    """Require authentication for API views."""
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            return JsonResponse(
                {'error': 'Authentication required'},
                status=401
            )
        return view_func(request, *args, **kwargs)
    return wrapper


# Usage
@api_login_required
def api_profile(request):
    return JsonResponse({
        'id': request.user.id,
        'username': request.user.username,
        'email': request.user.email
    })
```

Views decorated with `@api_login_required` return 401 for unauthenticated requests, letting the middleware handle token validation separately from view logic.

## Client Usage

The most secure approach for web apps is to have the server set tokens as httpOnly cookies:

```python
# Server-side: return token via httpOnly cookie
@csrf_exempt
def api_login(request):
    # ... authenticate user ...
    token, created = APIToken.objects.get_or_create(user=user, name='default')

    response = JsonResponse({
        'user': {'id': user.id, 'username': user.username}
    })
    response.set_cookie(
        'api_token',
        token.key,
        httponly=True,    # Cannot be read by JavaScript (XSS-safe)
        secure=True,      # HTTPS only in production
        samesite='Lax',   # CSRF protection
        max_age=86400,    # 24 hours
    )
    return response
```

```javascript
// Client-side: cookies are sent automatically
const response = await fetch('/api/login/', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  credentials: 'same-origin',  // Include cookies
  body: JSON.stringify({username: 'john', password: 'secret'})
});

// WARNING: Do NOT use localStorage for tokens!
// localStorage is vulnerable to XSS attacks.
// The httpOnly cookie above is automatically included in requests.

// Subsequent requests include the cookie automatically
const articles = await fetch('/api/articles/', {
  credentials: 'same-origin'
});
```

The `httponly=True` flag is critical: it prevents JavaScript from reading the cookie, making it immune to XSS attacks. The `credentials: 'same-origin'` option tells `fetch` to include cookies automatically.

## Common Pitfalls

1. **Storing tokens insecurely**: Never store tokens in localStorage for web apps. Use httpOnly cookies to prevent XSS attacks.

2. **No token expiration**: Tokens that never expire are a security risk. Always set an expiration date.

3. **Not using HTTPS**: Tokens sent over HTTP can be intercepted. Always use HTTPS in production.

## Best Practices

1. **Use cryptographically secure tokens**: Use `secrets.token_hex()` or `secrets.token_urlsafe()` for token generation.

2. **Set token expiration**: Tokens should have a reasonable lifetime (hours to days, not forever).

3. **Track token usage**: Record `last_used_at` for security auditing and inactive token cleanup.

4. **Support multiple tokens per user**: Allow users to have separate tokens for different devices or applications.

## Summary

Token authentication provides stateless API security by issuing unique tokens on login. Implement tokens with a database model, validate them via middleware, and protect views with authentication decorators. Always use secure storage (httpOnly cookies for web), set expiration dates, and use HTTPS in production.

## Code Examples

**Token model for API authentication with expiration support**

```python
import secrets
from django.db import models
from django.conf import settings

class APIToken(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    key = models.CharField(max_length=64, unique=True, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True)
    expires_at = models.DateTimeField(null=True, blank=True)

    def save(self, *args, **kwargs):
        if not self.key:
            self.key = secrets.token_hex(32)
        super().save(*args, **kwargs)

    @property
    def is_expired(self):
        if self.expires_at is None:
            return False
        from django.utils import timezone
        return timezone.now() > self.expires_at
```


## Resources

- [Django Authentication](https://docs.djangoproject.com/en/6.0/topics/auth/) — Official authentication documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*