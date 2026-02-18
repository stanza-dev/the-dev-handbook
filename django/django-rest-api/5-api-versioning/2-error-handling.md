---
source_course: "django-rest-api"
source_lesson: "django-rest-api-error-handling"
---

# Consistent Error Handling

Professional APIs provide consistent, informative error responses. Let's build a robust error handling system.

## Standard Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request data is invalid",
    "details": {
      "title": ["This field is required"],
      "email": ["Enter a valid email address"]
    }
  }
}
```

## Custom Exception Classes

```python
# exceptions.py
class APIException(Exception):
    """Base exception for API errors."""
    status_code = 500
    error_code = 'SERVER_ERROR'
    message = 'An error occurred'
    
    def __init__(self, message=None, details=None):
        self.message = message or self.message
        self.details = details or {}
        super().__init__(self.message)
    
    def to_dict(self):
        return {
            'error': {
                'code': self.error_code,
                'message': self.message,
                'details': self.details
            }
        }


class ValidationError(APIException):
    status_code = 400
    error_code = 'VALIDATION_ERROR'
    message = 'The request data is invalid'


class AuthenticationError(APIException):
    status_code = 401
    error_code = 'AUTHENTICATION_ERROR'
    message = 'Authentication required'


class PermissionDenied(APIException):
    status_code = 403
    error_code = 'PERMISSION_DENIED'
    message = 'You do not have permission to perform this action'


class NotFound(APIException):
    status_code = 404
    error_code = 'NOT_FOUND'
    message = 'The requested resource was not found'


class MethodNotAllowed(APIException):
    status_code = 405
    error_code = 'METHOD_NOT_ALLOWED'
    message = 'HTTP method not allowed'


class RateLimitExceeded(APIException):
    status_code = 429
    error_code = 'RATE_LIMIT_EXCEEDED'
    message = 'Too many requests'
```

## Exception Handling Middleware

```python
# middleware.py
import json
import logging
from django.http import JsonResponse
from .exceptions import APIException

logger = logging.getLogger(__name__)


class APIExceptionMiddleware:
    """Convert exceptions to JSON responses."""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        return self.get_response(request)
    
    def process_exception(self, request, exception):
        # Only handle API requests
        if not request.path.startswith('/api/'):
            return None
        
        if isinstance(exception, APIException):
            return JsonResponse(
                exception.to_dict(),
                status=exception.status_code
            )
        
        # Handle Django's built-in exceptions
        from django.core.exceptions import PermissionDenied as DjangoPermissionDenied
        from django.http import Http404
        
        if isinstance(exception, DjangoPermissionDenied):
            return JsonResponse(
                {'error': {'code': 'PERMISSION_DENIED', 'message': str(exception)}},
                status=403
            )
        
        if isinstance(exception, Http404):
            return JsonResponse(
                {'error': {'code': 'NOT_FOUND', 'message': 'Resource not found'}},
                status=404
            )
        
        # Log unexpected errors
        logger.exception('Unexpected API error')
        
        # Return generic error (don't expose details in production)
        from django.conf import settings
        if settings.DEBUG:
            message = str(exception)
        else:
            message = 'An unexpected error occurred'
        
        return JsonResponse(
            {'error': {'code': 'SERVER_ERROR', 'message': message}},
            status=500
        )
```

## Using Exceptions in Views

```python
from .exceptions import ValidationError, NotFound, PermissionDenied

def api_create_article(request):
    data = json.loads(request.body)
    
    # Validation
    errors = {}
    if not data.get('title'):
        errors['title'] = ['This field is required']
    if len(data.get('title', '')) > 200:
        errors['title'] = ['Title must be 200 characters or less']
    
    if errors:
        raise ValidationError(details=errors)
    
    article = Article.objects.create(
        title=data['title'],
        body=data.get('body', ''),
        author=request.user
    )
    
    return JsonResponse({'id': article.id}, status=201)


def api_article_detail(request, pk):
    try:
        article = Article.objects.get(pk=pk)
    except Article.DoesNotExist:
        raise NotFound(f'Article with id {pk} not found')
    
    if not article.is_published and article.author != request.user:
        raise PermissionDenied('You cannot view this draft article')
    
    return JsonResponse(serialize_article(article))
```

## Error Response Consistency

```python
# utils.py
def error_response(code, message, details=None, status=400):
    """Create a consistent error response."""
    return JsonResponse({
        'error': {
            'code': code,
            'message': message,
            'details': details or {}
        }
    }, status=status)


def success_response(data, status=200):
    """Create a consistent success response."""
    return JsonResponse(data, status=status)
```

## Resources

- [Django Exceptions](https://docs.djangoproject.com/en/6.0/ref/exceptions/) — Built-in Django exceptions reference

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*