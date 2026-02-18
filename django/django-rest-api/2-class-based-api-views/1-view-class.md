---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-view-class"
---

# Creating API View Classes

## Introduction

As your API grows, function-based views with long if/elif chains for HTTP methods become unwieldy. Class-based views (CBVs) offer a cleaner, object-oriented approach where each HTTP method maps to a class method. This separation makes code easier to read, test, and extend.

## Key Concepts

**Class-Based View (CBV)**: A view defined as a Python class that inherits from `django.views.View`. HTTP methods (GET, POST, etc.) are handled by corresponding methods (`get()`, `post()`).

**`dispatch()` method**: The entry point that routes requests to the appropriate handler method based on `request.method`.

**`as_view()` class method**: Converts the class into a callable view function for URL routing.

**Mixins**: Reusable classes that add functionality to views through multiple inheritance.

## Real World Context

In production Django applications, CBVs are preferred for APIs because:
- **Organization**: Each HTTP method has its own method—no more long if/elif chains
- **Reusability**: Common patterns can be extracted into mixins
- **Testability**: Individual methods can be tested in isolation
- **Extensibility**: Subclasses can override specific behaviors

## Deep Dive

### Basic API View Structure

```python
import json
from django.http import JsonResponse
from django.views import View
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator

@method_decorator(csrf_exempt, name='dispatch')
class ArticleAPIView(View):
    """RESTful API view for Article resource."""
    
    def get(self, request, pk=None):
        if pk:
            return self.retrieve(request, pk)
        return self.list(request)
    
    def post(self, request):
        return self.create(request)
    
    def put(self, request, pk):
        return self.update(request, pk)
    
    def delete(self, request, pk):
        return self.destroy(request, pk)
    
    # Implementation methods
    def list(self, request):
        articles = Article.objects.all()[:20]
        return JsonResponse({'data': list(articles.values('id', 'title'))})
    
    def retrieve(self, request, pk):
        try:
            article = Article.objects.get(pk=pk)
        except Article.DoesNotExist:
            return JsonResponse({'error': 'Not found'}, status=404)
        return JsonResponse({'id': article.id, 'title': article.title})
    
    def create(self, request):
        data = json.loads(request.body)
        article = Article.objects.create(title=data['title'])
        return JsonResponse({'id': article.id}, status=201)
    
    def update(self, request, pk):
        article = Article.objects.get(pk=pk)
        data = json.loads(request.body)
        article.title = data['title']
        article.save()
        return JsonResponse({'id': article.id})
    
    def destroy(self, request, pk):
        Article.objects.filter(pk=pk).delete()
        return JsonResponse({}, status=204)
```

### URL Configuration

```python
from django.urls import path
from .views import ArticleAPIView

urlpatterns = [
    path('api/articles/', ArticleAPIView.as_view(), name='article-list'),
    path('api/articles/<int:pk>/', ArticleAPIView.as_view(), name='article-detail'),
]
```

### Creating Reusable Mixins

```python
class JSONMixin:
    """Mixin for JSON handling."""
    
    def parse_json(self):
        try:
            return json.loads(self.request.body)
        except json.JSONDecodeError:
            return None
    
    def json_response(self, data, status=200):
        return JsonResponse(data, status=status)
    
    def error_response(self, message, status=400):
        return JsonResponse({'error': message}, status=status)

class CRUDMixin:
    """Mixin providing CRUD operations."""
    model = None
    fields = []
    
    def get_queryset(self):
        return self.model.objects.all()
    
    def serialize(self, obj):
        return {f: getattr(obj, f) for f in self.fields}
```

### Combining Mixins

```python
@method_decorator(csrf_exempt, name='dispatch')
class ArticleAPIView(JSONMixin, CRUDMixin, View):
    model = Article
    fields = ['id', 'title', 'body', 'created_at']
    
    def get(self, request, pk=None):
        if pk:
            obj = self.get_queryset().filter(pk=pk).first()
            if not obj:
                return self.error_response('Not found', 404)
            return self.json_response(self.serialize(obj))
        return self.json_response({'data': [self.serialize(o) for o in self.get_queryset()[:20]]})
```

## Common Pitfalls

1. **Forgetting `as_view()` in URLs**: `path('api/', MyView)` won't work. You must call `MyView.as_view()` to convert the class to a callable.

2. **Decorator placement**: Using `@csrf_exempt` directly on the class doesn't work. Use `@method_decorator(csrf_exempt, name='dispatch')` on the class.

3. **Not handling all methods**: If you define `get()` but not `post()`, POST requests return 405 automatically—which is correct behavior.

## Best Practices

1. **Separate concerns**: Put business logic in model methods or service classes, not in view methods.

2. **Use descriptive method names**: `list()`, `retrieve()`, `create()`, `update()`, `destroy()` are clearer than just `get()`, `post()`.

3. **Create a base API view**: Define common behavior once in a base class that all your API views inherit from.

4. **Keep mixins focused**: Each mixin should do one thing well (JSON handling, authentication, pagination).

## Summary

Class-based views organize API code by mapping HTTP methods to class methods. Use `@method_decorator` to apply decorators, call `.as_view()` in URL configuration, and extract reusable functionality into mixins. CBVs provide better organization, testability, and extensibility compared to function-based views for complex APIs.

## Resources

- [Class-based Views](https://docs.djangoproject.com/en/6.0/topics/class-based-views/) — Official guide to class-based views
- [Using Mixins with CBVs](https://docs.djangoproject.com/en/6.0/topics/class-based-views/mixins/) — How to use mixins with class-based views

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*