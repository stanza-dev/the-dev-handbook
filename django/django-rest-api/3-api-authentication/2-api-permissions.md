---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-permissions"
---

# API Permissions

Beyond authentication (who you are), permissions control what you can do. Implement fine-grained access control for your APIs.

## Permission Decorator

```python
from functools import wraps
from django.http import JsonResponse


def api_permission_required(permission):
    """Check if user has a specific permission."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return JsonResponse(
                    {'error': 'Authentication required'},
                    status=401
                )
            
            if not request.user.has_perm(permission):
                return JsonResponse(
                    {'error': 'Permission denied'},
                    status=403
                )
            
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator


# Usage
@api_permission_required('articles.add_article')
def api_create_article(request):
    # Only users with 'add_article' permission can access
    pass
```

## Object-Level Permissions

```python
class PermissionMixin:
    """Mixin for object-level permissions."""
    
    def has_object_permission(self, request, obj):
        """Override in subclasses."""
        return True
    
    def check_object_permission(self, request, obj):
        if not self.has_object_permission(request, obj):
            return JsonResponse({'error': 'Permission denied'}, status=403)
        return None


class ArticleAPIView(PermissionMixin, View):
    def has_object_permission(self, request, obj):
        """Users can only modify their own articles."""
        if request.method in ['PUT', 'PATCH', 'DELETE']:
            return obj.author == request.user or request.user.is_staff
        return True
    
    def put(self, request, pk):
        article = get_object_or_404(Article, pk=pk)
        
        # Check permission
        error = self.check_object_permission(request, article)
        if error:
            return error
        
        # Update article...
```

## Role-Based Access Control

```python
# models.py
class UserRole(models.Model):
    ROLES = [
        ('admin', 'Administrator'),
        ('editor', 'Editor'),
        ('author', 'Author'),
        ('viewer', 'Viewer'),
    ]
    
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    role = models.CharField(max_length=20, choices=ROLES, default='viewer')
    
    def has_role(self, *roles):
        return self.role in roles
```

```python
# decorators.py
def api_role_required(*roles):
    """Require specific user roles."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return JsonResponse({'error': 'Authentication required'}, status=401)
            
            try:
                user_role = request.user.userrole
            except UserRole.DoesNotExist:
                return JsonResponse({'error': 'No role assigned'}, status=403)
            
            if not user_role.has_role(*roles):
                return JsonResponse({'error': 'Insufficient privileges'}, status=403)
            
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator


# Usage
@api_role_required('admin', 'editor')
def api_publish_article(request, pk):
    # Only admins and editors can publish
    article = get_object_or_404(Article, pk=pk)
    article.is_published = True
    article.save()
    return JsonResponse({'status': 'published'})
```

## Permission Classes (More Flexible)

```python
class BasePermission:
    """Base class for permissions."""
    
    def has_permission(self, request, view):
        return True
    
    def has_object_permission(self, request, view, obj):
        return True


class IsAuthenticated(BasePermission):
    def has_permission(self, request, view):
        return request.user.is_authenticated


class IsAdminUser(BasePermission):
    def has_permission(self, request, view):
        return request.user.is_authenticated and request.user.is_staff


class IsOwnerOrReadOnly(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        return obj.author == request.user


class IsOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.author == request.user
```

```python
# Using permission classes in views
class ArticleAPIView(View):
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    
    def check_permissions(self, request):
        for permission_class in self.permission_classes:
            permission = permission_class()
            if not permission.has_permission(request, self):
                return JsonResponse({'error': 'Permission denied'}, status=403)
        return None
    
    def check_object_permissions(self, request, obj):
        for permission_class in self.permission_classes:
            permission = permission_class()
            if not permission.has_object_permission(request, self, obj):
                return JsonResponse({'error': 'Permission denied'}, status=403)
        return None
    
    def get(self, request, pk=None):
        error = self.check_permissions(request)
        if error:
            return error
        # ...
```

## Resources

- [Django Permissions](https://docs.djangoproject.com/en/6.0/topics/auth/default/#permissions-and-authorization) — Official guide to Django's permission system

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*