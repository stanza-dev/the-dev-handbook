---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-permissions"
---

# API Permissions

## Introduction

Authentication tells you WHO a user is. Permissions tell you WHAT they can do. A well-designed permission system prevents unauthorized access while keeping your code clean and maintainable.

Beyond authentication (who you are), permissions control what you can do. Implement fine-grained access control for your APIs.


## Key Concepts

**Authorization**: The process of determining whether an authenticated user has permission to perform a specific action.

**Object-Level Permissions**: Permissions that depend on the specific object being accessed (e.g., only the author can edit their article).

**Role-Based Access Control (RBAC)**: Assigning permissions based on user roles (admin, editor, viewer).

**Permission Classes**: Reusable classes that encapsulate permission logic for clean separation of concerns.

## Real World Context

Permissions are critical in:
- **Content management systems**: Authors, editors, and admins have different capabilities
- **Multi-tenant applications**: Users can only access their organization's data
- **E-commerce platforms**: Only order owners can view their order details
- **Social networks**: Privacy settings controlling who sees what

## Permission Decorator

This decorator checks Django's built-in permission system before allowing access. It distinguishes between authentication failures (401) and authorization failures (403):

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

Django auto-creates `add_`, `change_`, `delete_`, and `view_` permissions for each model, so `articles.add_article` is available without any extra configuration.

## Object-Level Permissions

Model-level permissions ("can create articles") don't cover ownership rules. Object-level permissions check whether a user can act on a specific instance:

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

The `has_object_permission()` method returns `True` for read requests but restricts write operations to the article's author or staff users. The permission check runs before any database modification.

## Role-Based Access Control

For applications with distinct user roles (admin, editor, viewer), store the role on a related model and check it with a decorator:

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

The `@api_role_required` decorator accepts multiple roles, so `('admin', 'editor')` means either role is sufficient. The `has_role()` method on `UserRole` handles the membership check.

## Permission Classes (More Flexible)

Permission classes encapsulate authorization logic into composable, reusable objects. Each class implements `has_permission()` for view-level checks and `has_object_permission()` for instance-level checks:

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

The `permission_classes` list is iterated in order -- all permissions must pass for the request to proceed. `IsOwnerOrReadOnly` allows GET requests from anyone but restricts modifications to the resource owner.

## Common Pitfalls

1. **Checking permissions too late**: Always check permissions before performing any database modifications or side effects.

2. **Forgetting object-level checks**: Model-level permissions (can create articles) don't prevent users from editing other users' articles.

3. **Mixing 401 and 403**: Return 401 for missing authentication and 403 for insufficient permissions. They mean different things.

## Best Practices

1. **Default to deny**: Require explicit permission grants rather than explicit denials.

2. **Use permission classes**: Encapsulate logic in reusable classes rather than inline checks.

3. **Test permissions thoroughly**: Write tests for both authorized and unauthorized access patterns.

4. **Separate authentication from authorization**: Keep the two concerns distinct in your code.

## Summary

API permissions control what authenticated users can do. Implement permission decorators for view-level access control, object-level permissions for resource ownership, and role-based access for organizational hierarchies. Always default to deny, check permissions early, and distinguish between 401 (not authenticated) and 403 (not authorized).

## Code Examples

**Permission decorator for protecting API endpoints**

```python
from functools import wraps
from django.http import JsonResponse

def api_permission_required(permission):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                return JsonResponse({'error': 'Authentication required'}, status=401)
            if not request.user.has_perm(permission):
                return JsonResponse({'error': 'Permission denied'}, status=403)
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator

@api_permission_required('articles.add_article')
def api_create_article(request):
    pass
```


## Resources

- [Django Permissions](https://docs.djangoproject.com/en/6.0/topics/auth/default/#permissions-and-authorization) — Official guide to Django's permission system

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*