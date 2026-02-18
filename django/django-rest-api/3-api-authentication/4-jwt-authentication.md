---
source_course: "django-rest-api"
source_lesson: "django-rest-api-jwt-authentication"
---

# JWT Authentication

## Introduction

JSON Web Tokens (JWT) are self-contained tokens that encode user information. Unlike server-stored tokens, JWTs can be verified without database lookups, making them popular for distributed systems and microservices.

## Key Concepts

**JWT (JSON Web Token)**: A compact, URL-safe token format consisting of three parts: header, payload, and signature.

**Access Token**: Short-lived JWT used to access protected resources (typically 15 minutes to 1 hour).

**Refresh Token**: Longer-lived token used to obtain new access tokens without re-authentication.

**Token Signature**: Cryptographic signature that ensures the token hasn't been tampered with.

## Real World Context

JWTs are ideal for:
- **Microservices**: Services can verify tokens without shared database access
- **Cross-domain APIs**: Tokens work across different domains
- **Mobile apps**: Tokens can be stored securely on device
- **Stateless scaling**: No server-side session storage needed

## Deep Dive

### JWT Structure

```
header.payload.signature

# Example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyX2lkIjoxLCJleHAiOjE3MDUzMjAwMDB9.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Simple JWT Implementation

```python
import jwt
import json
from datetime import datetime, timedelta
from django.conf import settings
from django.http import JsonResponse
from django.contrib.auth import authenticate

JWT_SECRET = settings.SECRET_KEY
JWT_ALGORITHM = 'HS256'
ACCESS_TOKEN_LIFETIME = timedelta(minutes=15)
REFRESH_TOKEN_LIFETIME = timedelta(days=7)

def generate_tokens(user):
    """Generate access and refresh tokens."""
    now = datetime.utcnow()
    
    access_payload = {
        'user_id': user.id,
        'username': user.username,
        'exp': now + ACCESS_TOKEN_LIFETIME,
        'iat': now,
        'type': 'access'
    }
    
    refresh_payload = {
        'user_id': user.id,
        'exp': now + REFRESH_TOKEN_LIFETIME,
        'iat': now,
        'type': 'refresh'
    }
    
    return {
        'access': jwt.encode(access_payload, JWT_SECRET, JWT_ALGORITHM),
        'refresh': jwt.encode(refresh_payload, JWT_SECRET, JWT_ALGORITHM),
    }

def verify_token(token, token_type='access'):
    """Verify and decode a JWT."""
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALGORITHM])
        if payload.get('type') != token_type:
            return None
        return payload
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None
```

### Auth Endpoints

```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def api_token_obtain(request):
    """Login and get tokens."""
    data = json.loads(request.body)
    user = authenticate(
        username=data.get('username'),
        password=data.get('password')
    )
    
    if not user:
        return JsonResponse({'error': 'Invalid credentials'}, status=401)
    
    tokens = generate_tokens(user)
    return JsonResponse(tokens)

@csrf_exempt
def api_token_refresh(request):
    """Get new access token using refresh token."""
    data = json.loads(request.body)
    refresh_token = data.get('refresh')
    
    payload = verify_token(refresh_token, 'refresh')
    if not payload:
        return JsonResponse({'error': 'Invalid refresh token'}, status=401)
    
    from django.contrib.auth import get_user_model
    User = get_user_model()
    
    try:
        user = User.objects.get(id=payload['user_id'])
    except User.DoesNotExist:
        return JsonResponse({'error': 'User not found'}, status=401)
    
    tokens = generate_tokens(user)
    return JsonResponse(tokens)
```

### JWT Authentication Middleware

```python
class JWTAuthMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        auth = request.headers.get('Authorization', '')
        
        if auth.startswith('Bearer '):
            token = auth[7:]
            payload = verify_token(token, 'access')
            
            if payload:
                from django.contrib.auth import get_user_model
                User = get_user_model()
                try:
                    request.user = User.objects.get(id=payload['user_id'])
                except User.DoesNotExist:
                    pass
        
        return self.get_response(request)
```

## Common Pitfalls

1. **Storing JWTs in localStorage**: Vulnerable to XSS attacks. Use httpOnly cookies for web apps.

2. **No token revocation**: JWTs are valid until expiry. Implement a token blacklist for logout.

3. **Long-lived access tokens**: Keep access tokens short (15 min). Use refresh tokens for longer sessions.

## Best Practices

1. **Use short-lived access tokens**: 15 minutes is a good balance.

2. **Implement refresh token rotation**: Issue new refresh token with each refresh.

3. **Store minimal data in payload**: Only user ID, not sensitive information.

4. **Use HTTPS always**: Tokens are credentials—protect them in transit.

## Summary

JWTs provide stateless authentication suitable for distributed systems. They consist of a header, payload, and signature. Implement short-lived access tokens with refresh tokens for security. Be aware of storage security—httpOnly cookies are safer than localStorage for web applications.

## Resources

- [PyJWT Documentation](https://pyjwt.readthedocs.io/en/stable/) — Python JWT library documentation
- [Django Authentication](https://docs.djangoproject.com/en/6.0/topics/auth/) — Django authentication system

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*