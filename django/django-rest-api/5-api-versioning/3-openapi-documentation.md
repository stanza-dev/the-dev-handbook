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

An OpenAPI spec is a structured dictionary describing your API's endpoints, parameters, request bodies, and response schemas:

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

The `$ref` values point to reusable component schemas, keeping the spec DRY when the same data structure appears in multiple endpoints.

### Serving the Schema

Expose the spec as a JSON endpoint that tools like Swagger UI and client generators can consume:

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

Clients and documentation tools fetch this endpoint to discover your API's structure automatically.

### Auto-Generating Schema from Views (Pure Django)

You can build a schema generator that inspects your URL configuration and produces a valid OpenAPI spec without any third-party REST framework:

```python
# schema_generator.py
import json
import inspect
from django.urls import get_resolver, URLPattern
from django.http import JsonResponse

def generate_openapi_schema(request):
    """Auto-generate OpenAPI schema from URL patterns."""
    schema = {
        'openapi': '3.0.0',
        'info': {
            'title': 'My API',
            'version': '1.0.0',
            'description': 'Auto-generated API documentation',
        },
        'paths': {},
        'servers': [
            {'url': request.build_absolute_uri('/'), 'description': 'Current server'}
        ],
    }

    resolver = get_resolver()
    for pattern in resolver.url_patterns:
        if hasattr(pattern, 'url_patterns'):
            for sub in pattern.url_patterns:
                if isinstance(sub, URLPattern) and hasattr(sub.callback, 'schema_info'):
                    path = '/' + str(pattern.pattern) + str(sub.pattern)
                    schema['paths'][path] = sub.callback.schema_info

    return JsonResponse(schema)

# Decorator to attach schema metadata to views
def api_schema(method, summary, parameters=None, request_body=None, responses=None):
    def decorator(view_func):
        if not hasattr(view_func, 'schema_info'):
            view_func.schema_info = {}
        view_func.schema_info[method.lower()] = {
            'summary': summary,
            'parameters': parameters or [],
            'responses': responses or {'200': {'description': 'Success'}},
        }
        if request_body:
            view_func.schema_info[method.lower()]['requestBody'] = request_body
        return view_func
    return decorator
```

The `@api_schema` decorator attaches metadata to each view function. The generator then walks all URL patterns and collects this metadata into a valid OpenAPI document.

### Serving Swagger UI with Static Files

You can serve the Swagger UI directly as a Django template:

```python
# views.py
from django.shortcuts import render

def swagger_ui(request):
    """Serve Swagger UI pointing to our schema endpoint."""
    return render(request, 'swagger_ui.html', {
        'schema_url': '/api/schema/',
    })

# urls.py
urlpatterns = [
    path('api/schema/', generate_openapi_schema, name='api-schema'),
    path('api/docs/', swagger_ui, name='api-docs'),
]
```

The template loads Swagger UI from a CDN and points it at your `/api/schema/` endpoint, giving developers an interactive interface to explore and test your API.

## Common Pitfalls

1. **Outdated documentation**: Auto-generate docs from code when possible to keep them in sync.

2. **Missing examples**: Include request/response examples for clarity.

3. **Exposing internal APIs**: Don't document internal endpoints meant only for your services.

## Best Practices

1. **Auto-generate when possible**: Use schema generators or decorators to keep docs in sync with code.
2. **Include examples**: Show real request/response payloads.
3. **Document errors**: Show possible error responses and their meanings.
4. **Version your docs**: Documentation should match the API version.

## Summary

OpenAPI provides a standardized way to document your API. Use auto-generation tools to keep docs in sync with code, include examples and error documentation, and serve interactive docs with Swagger UI for easy testing.

## Code Examples

**Serving a manually defined OpenAPI schema as a JSON endpoint**

```python
from django.http import JsonResponse

OPENAPI_SCHEMA = {
    'openapi': '3.0.0',
    'info': {'title': 'My API', 'version': '1.0.0'},
    'paths': {
        '/api/articles/': {
            'get': {
                'summary': 'List articles',
                'responses': {'200': {'description': 'List of articles'}}
            }
        }
    }
}

def openapi_schema(request):
    return JsonResponse(OPENAPI_SCHEMA)
```


## Resources

- [OpenAPI Specification](https://swagger.io/specification/) — Official OpenAPI specification

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*