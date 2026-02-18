---
source_course: "django-rest-api"
source_lesson: "django-rest-api-openapi-documentation"
---

# OpenAPI Documentation

## Introduction

Good documentation is as important as good code. OpenAPI (formerly Swagger) is the industry standard for describing REST APIs. It enables automatic documentation generation, client SDK generation, and API testing.

## Key Concepts

**OpenAPI Specification**: A standard format (JSON/YAML) for describing REST API structure, endpoints, parameters, and responses.

**Swagger UI**: An interactive documentation interface generated from OpenAPI specs.

**Schema**: The structure definition of request/response bodies.

## Real World Context

In production environments, OpenAPI documentation:
- Enables frontend developers to work in parallel with backend
- Auto-generates client SDKs in multiple languages
- Provides interactive testing without external tools
- Serves as a contract between teams

## Deep Dive

### Manual OpenAPI Schema

```python
# openapi.py
OPENAPI_SCHEMA = {
    'openapi': '3.0.0',
    'info': {
        'title': 'My API',
        'version': '1.0.0',
        'description': 'API for managing articles'
    },
    'paths': {
        '/api/articles/': {
            'get': {
                'summary': 'List articles',
                'parameters': [
                    {'name': 'page', 'in': 'query', 'schema': {'type': 'integer'}},
                    {'name': 'search', 'in': 'query', 'schema': {'type': 'string'}}
                ],
                'responses': {
                    '200': {
                        'description': 'List of articles',
                        'content': {
                            'application/json': {
                                'schema': {'$ref': '#/components/schemas/ArticleList'}
                            }
                        }
                    }
                }
            },
            'post': {
                'summary': 'Create article',
                'requestBody': {
                    'content': {
                        'application/json': {
                            'schema': {'$ref': '#/components/schemas/ArticleCreate'}
                        }
                    }
                },
                'responses': {
                    '201': {'description': 'Article created'}
                }
            }
        }
    },
    'components': {
        'schemas': {
            'Article': {
                'type': 'object',
                'properties': {
                    'id': {'type': 'integer'},
                    'title': {'type': 'string'},
                    'body': {'type': 'string'},
                    'created_at': {'type': 'string', 'format': 'date-time'}
                }
            }
        }
    }
}
```

### Serving the Schema

```python
# views.py
from django.http import JsonResponse
from .openapi import OPENAPI_SCHEMA

def openapi_schema(request):
    return JsonResponse(OPENAPI_SCHEMA)

# urls.py
urlpatterns = [
    path('api/schema/', openapi_schema, name='api-schema'),
]
```

### Using drf-spectacular (Recommended)

```python
# pip install drf-spectacular

# settings.py
INSTALLED_APPS = [
    'drf_spectacular',
]

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

SPECTACULAR_SETTINGS = {
    'TITLE': 'My API',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
}

# urls.py
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

urlpatterns = [
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
]
```

## Common Pitfalls

1. **Outdated documentation**: Auto-generate docs from code when possible to keep them in sync.

2. **Missing examples**: Include request/response examples for clarity.

3. **Exposing internal APIs**: Don't document internal endpoints meant only for your services.

## Best Practices

1. **Auto-generate when possible**: Use tools like drf-spectacular to generate docs from code.
2. **Include examples**: Show real request/response payloads.
3. **Document errors**: Show possible error responses and their meanings.
4. **Version your docs**: Documentation should match the API version.

## Summary

OpenAPI provides a standardized way to document your API. Use auto-generation tools to keep docs in sync with code, include examples and error documentation, and serve interactive docs with Swagger UI for easy testing.

## Resources

- [OpenAPI Specification](https://swagger.io/specification/) — Official OpenAPI specification

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*