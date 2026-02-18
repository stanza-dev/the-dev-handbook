---
source_course: "django-rest-api"
source_lesson: "django-rest-api-token-authentication"
---

# Token-Based Authentication

Token authentication is the standard for API security. Instead of sending credentials with every request, clients receive a token after login.

## How Token Auth Works

```
1. Client sends username/password to login endpoint
2. Server validates credentials and returns a token
3. Client stores the token
4. Client sends token in Authorization header with every request
5. Server validates token and identifies the user
```

## Creating a Token Model

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

## Login Endpoint

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

## Token Authentication Middleware

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

```python
# settings.py
MIDDLEWARE = [
    # ... other middleware
    'myapp.middleware.TokenAuthMiddleware',
]
```

## Authentication Decorator

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

## Client Usage

```javascript
// Login
const response = await fetch('/api/login/', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({username: 'john', password: 'secret'})
});
const {token} = await response.json();

// Store token (localStorage, secure storage, etc.)
localStorage.setItem('api_token', token);

// Use token in subsequent requests
const articles = await fetch('/api/articles/', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});
```

## Resources

- [Django Authentication](https://docs.djangoproject.com/en/6.0/topics/auth/) — Official authentication documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*