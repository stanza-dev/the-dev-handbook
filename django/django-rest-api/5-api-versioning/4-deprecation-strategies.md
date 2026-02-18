---
source_course: "django-rest-api"
source_lesson: "django-rest-api-deprecation-strategies"
---

# API Deprecation Strategies

## Introduction

APIs evolve. When you need to make breaking changes, a thoughtful deprecation strategy keeps existing clients working while guiding them to newer versions.

## Key Concepts

**Deprecation**: Marking a feature as outdated and scheduled for removal.

**Sunset Header**: HTTP header indicating when an API will be removed.

**Breaking Change**: A change that requires clients to update their code.

**Migration Period**: Time given to clients to switch to the new version.

## Deep Dive

### Deprecation Headers

```python
from functools import wraps
from django.http import JsonResponse

def deprecated(sunset_date, successor_url=None):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            response = view_func(request, *args, **kwargs)
            
            response['Deprecation'] = 'true'
            response['Sunset'] = sunset_date
            
            if successor_url:
                response['Link'] = f'<{successor_url}>; rel="successor-version"'
            
            return response
        return wrapper
    return decorator

@deprecated(sunset_date='2027-01-01T00:00:00Z', successor_url='/api/v2/articles/')
def api_v1_articles(request):
    # Old endpoint
    pass
```

### Versioned Response Fields

```python
def api_user(request, pk):
    user = User.objects.get(pk=pk)
    
    response = {
        'id': user.id,
        'username': user.username,
        # New field
        'display_name': user.get_full_name(),
        # Deprecated field (kept for compatibility)
        'name': user.get_full_name(),  # DEPRECATED: Use display_name
    }
    
    return JsonResponse(response)
```

### Communication Strategy

```python
# Include deprecation warnings in responses
def api_response_with_warnings(data, warnings=None):
    response = {'data': data}
    if warnings:
        response['_warnings'] = warnings
    return JsonResponse(response)

def api_articles(request):
    data = get_articles()
    return api_response_with_warnings(
        data,
        warnings=['This endpoint will be removed on 2027-01-01. Use /api/v2/articles/']
    )
```

## Best Practices

1. **Announce early**: Give at least 6-12 months notice for major changes.
2. **Use standard headers**: Deprecation and Sunset headers are machine-readable.
3. **Provide migration guides**: Document how to switch to the new version.
4. **Monitor usage**: Track deprecated endpoint usage to inform timeline.
5. **Gradual rollout**: Release new versions before deprecating old ones.

## Summary

Deprecation requires clear communication through HTTP headers, documentation, and direct outreach. Provide adequate migration time, document changes thoroughly, and monitor usage to ensure smooth transitions.

## Resources

- [HTTP Sunset Header](https://datatracker.ietf.org/doc/html/rfc8594) — RFC 8594 - The Sunset HTTP Header Field

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*