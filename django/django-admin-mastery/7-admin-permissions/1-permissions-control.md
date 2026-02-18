---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-permissions-control"
---

# Admin Permissions

Django provides fine-grained control over what users can do in the admin interface.

## Permission Methods

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

## Custom Permissions

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

## Field-Level Permissions

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

## Object-Level Permissions

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

## Group-Based Access

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

## Resources

- [Admin Permissions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.has_add_permission) — ModelAdmin permission methods

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*