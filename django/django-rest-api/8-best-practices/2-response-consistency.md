---
source_course: "django-rest-api"
source_lesson: "django-rest-api-response-consistency"
---

# Consistent Response Formats

## Introduction

Inconsistent API responses frustrate developers and create bugs. A standardized response format across all endpoints makes your API predictable and professional.

## Key Concepts

**Response Envelope**: A wrapper structure that contains data, metadata, and errors consistently.

**Error Schema**: Standardized format for all error responses.

**Metadata**: Information about the response (pagination, request ID, timing).

## Deep Dive

### Standard Success Response

```python
# utils/responses.py
from django.http import JsonResponse
import time

def api_response(data=None, meta=None, status=200):
    response = {
        'success': True,
        'data': data,
    }
    if meta:
        response['meta'] = meta
    return JsonResponse(response, status=status)

# Usage
def get_article(request, pk):
    article = Article.objects.get(pk=pk)
    return api_response(
        data={'id': article.id, 'title': article.title},
        meta={'request_id': request.id}
    )
```

### Standard Error Response

```python
def api_error(code, message, details=None, status=400):
    response = {
        'success': False,
        'error': {
            'code': code,
            'message': message,
        }
    }
    if details:
        response['error']['details'] = details
    return JsonResponse(response, status=status)

# Usage
def create_article(request):
    if not data.get('title'):
        return api_error(
            code='VALIDATION_ERROR',
            message='Validation failed',
            details={'title': ['This field is required']},
            status=400
        )
```

### Pagination Response

```python
def paginated_response(items, page, page_size, total):
    return api_response(
        data=items,
        meta={
            'pagination': {
                'page': page,
                'page_size': page_size,
                'total_items': total,
                'total_pages': (total + page_size - 1) // page_size,
            }
        }
    )
```

## Best Practices

1. **Always include success indicator**: Makes error checking easy.
2. **Use error codes**: Machine-readable codes enable programmatic handling.
3. **Include request IDs**: Helps with debugging and support.
4. **Document your format**: Include schema in API documentation.

## Summary

Consistent response formats make APIs predictable. Use a standard envelope with success indicator, data/error separation, and metadata for pagination and debugging.

## Resources

- [JSON:API Specification](https://jsonapi.org/) — A specification for building APIs in JSON

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*