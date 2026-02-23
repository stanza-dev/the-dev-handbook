---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-permissions-control"
---

# Admin Permissions

## Introduction

Django's admin provides fine-grained permission control at the model, object, and field levels. In this lesson, you will learn how to use the has_*_permission methods, custom permissions, and queryset filtering to build a secure admin interface.

## Key Concepts

**has_add_permission**: Controls whether the user can create new objects.

**has_change_permission**: Controls whether the user can edit objects (obj=None for list access).

**has_delete_permission**: Controls whether the user can delete objects.

**has_view_permission**: Controls read-only access to objects.

**has_module_permission**: Controls whether the app appears on the admin index.

**get_queryset()**: Filters which objects are visible to the user.

## Real World Context

A news organization has reporters who can only edit their own drafts, editors who can edit any article, and publishers who can change an article's status to 'published'. By implementing has_change_permission with object-level checks and custom 'publish' permissions, the admin enforces the editorial workflow without any external authorization library.

## Deep Dive

Django provides fine-grained control over what users can do in the admin interface.

### Permission Methods

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def has_add_permission(self, request):
        """Can user add new articles?"""
        return request.user.has_perm('blog.add_article')
    
    def has_change_permission(self, request, obj=None):
        """Can user edit articles?"""
        if obj is None:
            # Checking for list view access
            return request.user.has_perm('blog.change_article')
        # Checking for specific object
        return obj.author == request.user or request.user.is_superuser
    
    def has_delete_permission(self, request, obj=None):
        """Can user delete articles?"""
        if obj and obj.status == 'published':
            # Can't delete published articles
            return False
        return request.user.has_perm('blog.delete_article')
    
    def has_view_permission(self, request, obj=None):
        """Can user view articles?"""
        return True  # All staff can view
    
    def has_module_permission(self, request):
        """Show this app in admin index?"""
        return request.user.has_perm('blog.view_article')
```

### Custom Permissions

```python
# models.py
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    
    class Meta:
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature articles'),
            ('view_statistics', 'Can view article statistics'),
        ]
```

```python
# admin.py
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['publish', 'feature']
    
    @admin.action(description='Publish selected articles')
    def publish(self, request, queryset):
        if not request.user.has_perm('blog.publish_article'):
            self.message_user(
                request,
                'You do not have permission to publish.',
                messages.ERROR
            )
            return
        queryset.update(status='published')
    
    def get_actions(self, request):
        actions = super().get_actions(request)
        if not request.user.has_perm('blog.feature_article'):
            if 'feature' in actions:
                del actions['feature']
        return actions
```

### Field-Level Permissions

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_fields(self, request, obj=None):
        fields = ['title', 'body', 'status', 'author', 'pub_date']
        
        # Only superusers see the author field
        if not request.user.is_superuser:
            fields.remove('author')
        
        return fields
    
    def get_readonly_fields(self, request, obj=None):
        readonly = ['created_at', 'updated_at']
        
        # Non-superusers can't change status of published articles
        if obj and obj.status == 'published' and not request.user.is_superuser:
            readonly.append('status')
        
        return readonly
```

### Object-Level Permissions

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.is_superuser:
            return qs
        # Users only see their own articles
        return qs.filter(author=request.user)
    
    def save_model(self, request, obj, form, change):
        if not change:  # Creating new object
            obj.author = request.user
        super().save_model(request, obj, form, change)
```

### Group-Based Access

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def has_change_permission(self, request, obj=None):
        # Editors can change any article
        if request.user.groups.filter(name='Editors').exists():
            return True
        # Authors can only change their own
        if obj and obj.author == request.user:
            return True
        return request.user.is_superuser
    
    def get_list_filter(self, request):
        filters = ['status', 'pub_date']
        # Only editors see the author filter
        if request.user.groups.filter(name='Editors').exists():
            filters.append('author')
        return filters
```

## Common Pitfalls

1. **Not handling obj=None in permission methods**: has_change_permission is called twice: once with obj=None (for list view access) and once with the actual object (for the change form). Returning False when obj is None blocks the entire change list.

2. **Relying only on get_queryset() for security**: Filtering the queryset hides objects from the list view, but a user who knows the object's ID can still access it via direct URL unless has_change_permission also checks ownership.

3. **Forgetting to call super()**: Not calling super() in permission methods bypasses Django's default permission framework, including the is_staff check.

## Best Practices

1. **Layer permissions from broad to narrow**: Start with has_module_permission (can the user see this app?), then has_view/change_permission (can they view/edit?), then get_queryset (which objects?), then field-level restrictions.

2. **Use get_readonly_fields() for field-level control**: Instead of hiding fields entirely, make them read-only for restricted users so they can still see the information.

3. **Auto-assign ownership in save_model()**: When creating new objects, set the author/owner to request.user so object-level permissions work from the first save.

## Summary

- Django admin provides five permission methods: has_add/change/delete/view/module_permission.
- Permission methods receive obj=None for list-level checks and the instance for object-level checks.
- Combine get_queryset() with permission methods for proper security.
- Custom permissions defined in Model Meta enable fine-grained action control.
- Use get_readonly_fields() and get_fields() for field-level permission control.

## Code Examples

**Object-level permissions: authors can edit their own articles, published articles cannot be deleted**

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def has_change_permission(self, request, obj=None):
        if obj is None:
            return request.user.has_perm('blog.change_article')
        return obj.author == request.user or request.user.is_superuser

    def has_delete_permission(self, request, obj=None):
        if obj and obj.status == 'published':
            return False
        return super().has_delete_permission(request, obj)
```


## Resources

- [Admin Permissions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.has_add_permission) — ModelAdmin permission methods

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*