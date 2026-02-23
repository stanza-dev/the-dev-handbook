---
source_course: "django-rest-api"
source_lesson: "django-rest-api-error-handling"
---

# Consistent Error Handling

## Introduction

When something goes wrong, your API's error response is the only information the developer has to diagnose the problem. Consistent, informative error responses are the difference between a 5-minute fix and hours of frustrating debugging.

Professional APIs provide consistent, informative error responses. Let's build a robust error handling system.


## Key Concepts

**Error Code**: A machine-readable string (e.g., `VALIDATION_ERROR`) that clients can use for programmatic error handling.

**Error Message**: A human-readable description of what went wrong.

**Error Details**: Field-level validation errors or additional context.

**Exception Middleware**: A Django middleware that catches exceptions and converts them to consistent JSON error responses.


## Real World Context

Consistent error handling is essential for:
- **Developer experience**: Clear errors reduce support tickets and integration time
- **Monitoring**: Structured error codes enable automated alerting and tracking
- **Debugging**: Detailed error responses speed up issue resolution
- **Client reliability**: Predictable error formats allow robust client-side error handling

## Standard Error Format

A well-structured error response includes a machine-readable code, a human-readable message, and optional field-level details:

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

This structure lets clients handle errors programmatically via `code`, display messages to users via `message`, and highlight specific form fields via `details`.

## Custom Exception Classes

Define a hierarchy of exception classes that map to HTTP status codes. Each exception knows its status code and error code, producing consistent responses:

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

Each subclass overrides `status_code`, `error_code`, and `message` with sensible defaults. The `to_dict()` method produces the standard error envelope shown above.

## Exception Handling Middleware

This middleware catches exceptions from any API view and converts them into consistent JSON responses, eliminating try/except boilerplate from individual views:

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
```

The `process_exception` hook only fires for `/api/` paths, leaving Django's default error handling for non-API routes. In production (`DEBUG=False`), internal details are hidden from clients.

## Using Exceptions in Views

With the middleware in place, views simply raise exceptions and the middleware handles the response formatting:

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

Raising `NotFound` or `ValidationError` is cleaner than manually constructing error responses in every view. The middleware ensures consistent formatting across all endpoints.

## Error Response Consistency

For views that prefer to return responses directly instead of raising exceptions, these helper functions enforce the same consistent format:

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

Using `error_response()` and `success_response()` everywhere ensures that every endpoint produces the same JSON structure, whether handling success or failure.

## Common Pitfalls

1. **Exposing stack traces in production**: Never return Python tracebacks to clients. They reveal internal implementation details and security vulnerabilities.

2. **Inconsistent error formats**: Mixing `{'error': 'msg'}` and `{'message': 'msg'}` across endpoints forces clients to handle multiple formats.

3. **Generic error messages**: `{'error': 'Something went wrong'}` is useless. Be specific about what failed and why.

## Best Practices

1. **Use a standard error envelope**: `{error: {code, message, details}}` across all endpoints.

2. **Use exception middleware**: Catch exceptions centrally instead of try/except in every view.

3. **Log server errors**: Log 500 errors with full context for debugging, but return sanitized messages to clients.

4. **Include request IDs**: Add unique request IDs to error responses for easier support and debugging.

## Summary

Consistent error handling builds trust with API consumers. Define custom exception classes for common error types, use exception middleware for centralized handling, and always return structured error responses with machine-readable codes, human-readable messages, and field-level details.

## Code Examples

**Custom exception classes for consistent API error responses**

```python
class APIException(Exception):
    status_code = 500
    error_code = 'SERVER_ERROR'
    message = 'An error occurred'

    def __init__(self, message=None, details=None):
        self.message = message or self.message
        self.details = details or {}

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

class NotFound(APIException):
    status_code = 404
    error_code = 'NOT_FOUND'
    message = 'The requested resource was not found'
```


## Resources

- [Django Exceptions](https://docs.djangoproject.com/en/6.0/ref/exceptions/) — Built-in Django exceptions reference

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*