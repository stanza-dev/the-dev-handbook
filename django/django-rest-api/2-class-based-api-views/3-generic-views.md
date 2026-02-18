---
source_course: "django-rest-api"
source_lesson: "django-rest-api-generic-views"
---

# Generic API Views

## Introduction

Writing the same CRUD logic for every model gets repetitive. Django provides generic class-based views that handle common patterns with minimal code. Combined with mixins, you can build powerful, DRY API endpoints.

## Key Concepts

**Generic Views**: Pre-built view classes that handle common patterns like displaying lists, showing details, or handling forms.

**ListView**: Returns a list of objects from a queryset.

**DetailView**: Returns a single object by primary key or slug.

**Mixins**: Reusable classes that add specific functionality (create, update, delete) to views.

## Real World Context

In large applications, you might have dozens of models, each needing similar CRUD endpoints. Generic views:
- Reduce code duplication significantly
- Ensure consistent behavior across endpoints
- Make it easy to add features like pagination globally
- Simplify testing—test the base classes once

## Deep Dive

### Building a Generic API View Base

```python
from django.views import View
from django.http import JsonResponse
import json

class APIView(View):
    """Base class for all API views."""
    
    def dispatch(self, request, *args, **kwargs):
        try:
            return super().dispatch(request, *args, **kwargs)
        except Exception as e:
            return JsonResponse({'error': str(e)}, status=500)
    
    def parse_json(self):
        try:
            return json.loads(self.request.body)
        except json.JSONDecodeError:
            return None
```

### List and Detail Mixins

```python
class ListMixin:
    """Provides list() method for collections."""
    model = None
    list_fields = ['id']
    page_size = 20
    
    def get_queryset(self):
        return self.model.objects.all()
    
    def list(self, request):
        qs = self.get_queryset()[:self.page_size]
        data = list(qs.values(*self.list_fields))
        return JsonResponse({'data': data, 'count': len(data)})

class DetailMixin:
    """Provides retrieve() method for single objects."""
    model = None
    detail_fields = None
    
    def get_object(self, pk):
        return self.model.objects.get(pk=pk)
    
    def retrieve(self, request, pk):
        try:
            obj = self.get_object(pk)
        except self.model.DoesNotExist:
            return JsonResponse({'error': 'Not found'}, status=404)
        
        fields = self.detail_fields or [f.name for f in self.model._meta.fields]
        data = {f: getattr(obj, f) for f in fields}
        return JsonResponse(data)
```

### Create, Update, Delete Mixins

```python
class CreateMixin:
    """Provides create() method."""
    model = None
    create_fields = []
    
    def create(self, request):
        data = self.parse_json()
        if not data:
            return JsonResponse({'error': 'Invalid JSON'}, status=400)
        
        filtered = {k: v for k, v in data.items() if k in self.create_fields}
        obj = self.model.objects.create(**filtered)
        return JsonResponse({'id': obj.id}, status=201)

class UpdateMixin:
    """Provides update() method."""
    model = None
    update_fields = []
    
    def update(self, request, pk):
        try:
            obj = self.model.objects.get(pk=pk)
        except self.model.DoesNotExist:
            return JsonResponse({'error': 'Not found'}, status=404)
        
        data = self.parse_json()
        for field in self.update_fields:
            if field in data:
                setattr(obj, field, data[field])
        obj.save()
        return JsonResponse({'id': obj.id})

class DeleteMixin:
    """Provides destroy() method."""
    model = None
    
    def destroy(self, request, pk):
        deleted, _ = self.model.objects.filter(pk=pk).delete()
        if not deleted:
            return JsonResponse({'error': 'Not found'}, status=404)
        return JsonResponse({}, status=204)
```

### Combining into a Complete View

```python
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

@method_decorator(csrf_exempt, name='dispatch')
class ModelAPIView(APIView, ListMixin, DetailMixin, CreateMixin, UpdateMixin, DeleteMixin):
    """Complete CRUD API view."""
    
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

# Usage - minimal code needed!
class ArticleAPIView(ModelAPIView):
    model = Article
    list_fields = ['id', 'title', 'created_at']
    detail_fields = ['id', 'title', 'body', 'created_at']
    create_fields = ['title', 'body']
    update_fields = ['title', 'body']
```

## Common Pitfalls

1. **Mixin order matters**: In Python, mixins are resolved left-to-right. Place more specific mixins first.

2. **Forgetting to set model**: Each view must define the `model` class attribute.

3. **Over-abstracting**: Don't create complex inheritance hierarchies for simple use cases.

## Best Practices

1. **Start simple**: Build views manually first, then extract patterns into mixins.

2. **One mixin, one job**: Each mixin should provide a single piece of functionality.

3. **Override hooks, not methods**: Provide `get_queryset()`, `get_object()` hooks rather than overriding entire methods.

4. **Document your base classes**: Team members need to know what attributes to set.

## Summary

Generic views and mixins enable DRY API development by extracting common CRUD patterns into reusable components. Build a base `APIView` with error handling, create focused mixins for each operation (List, Detail, Create, Update, Delete), then compose them into complete views. This approach reduces boilerplate while maintaining flexibility.

## Resources

- [Generic Display Views](https://docs.djangoproject.com/en/6.0/ref/class-based-views/generic-display/) — ListView and DetailView reference
- [Built-in Mixins Reference](https://docs.djangoproject.com/en/6.0/ref/class-based-views/mixins/) — All available Django mixins

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*