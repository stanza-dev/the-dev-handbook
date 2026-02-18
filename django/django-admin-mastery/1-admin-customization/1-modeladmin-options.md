---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-modeladmin-options"
---

# ModelAdmin Configuration

The ModelAdmin class provides extensive options for customizing how models appear in the admin interface.

## Basic ModelAdmin

```python
from django.contrib import admin
from .models import Article


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    # List view options
    list_display = ['title', 'author', 'status', 'pub_date', 'views']
    list_display_links = ['title']  # Which fields link to change page
    list_editable = ['status']       # Edit directly in list view
    list_filter = ['status', 'pub_date', 'author']
    list_per_page = 25
    list_max_show_all = 200
    
    # Search
    search_fields = ['title', 'body', 'author__username']
    search_help_text = 'Search by title, body, or author name'
    
    # Ordering
    ordering = ['-pub_date']
    
    # Date navigation
    date_hierarchy = 'pub_date'
```

## Display Options

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'status_badge', 'word_count', 'pub_date']
    
    @admin.display(description='Status', ordering='status')
    def status_badge(self, obj):
        """Display status with color."""
        colors = {
            'draft': 'gray',
            'published': 'green',
            'archived': 'red',
        }
        color = colors.get(obj.status, 'black')
        return format_html(
            '<span style="color: {};">{}</span>',
            color,
            obj.get_status_display()
        )
    
    @admin.display(description='Words')
    def word_count(self, obj):
        """Calculate word count."""
        return len(obj.body.split())
```

## Change Form Configuration

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    # Fields and layout
    fields = ['title', 'slug', 'author', ('status', 'pub_date'), 'body', 'tags']
    # Or use fieldsets for sections
    fieldsets = [
        (None, {
            'fields': ['title', 'slug', 'body']
        }),
        ('Publishing', {
            'fields': ['author', 'status', 'pub_date'],
            'classes': ['collapse'],  # Collapsible section
        }),
        ('Metadata', {
            'fields': ['tags', 'featured_image'],
            'classes': ['wide'],
        }),
    ]
    
    # Read-only fields
    readonly_fields = ['created_at', 'updated_at', 'views']
    
    # Raw ID for foreign keys (popup instead of dropdown)
    raw_id_fields = ['author']
    
    # Autocomplete (requires search_fields on related model)
    autocomplete_fields = ['tags']
    
    # Filter horizontal for many-to-many
    filter_horizontal = ['tags']  # Or filter_vertical
    
    # Prepopulate slug from title
    prepopulated_fields = {'slug': ('title',)}
```

## Permission Control

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def has_add_permission(self, request):
        return request.user.has_perm('blog.add_article')
    
    def has_change_permission(self, request, obj=None):
        if obj is None:
            return True
        # Users can only edit their own articles
        return obj.author == request.user or request.user.is_superuser
    
    def has_delete_permission(self, request, obj=None):
        return request.user.is_superuser
    
    def has_view_permission(self, request, obj=None):
        return True
```

## Filtering Querysets

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.is_superuser:
            return qs
        # Non-superusers only see their own articles
        return qs.filter(author=request.user)
    
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == 'author':
            # Only show active users in author dropdown
            kwargs['queryset'] = User.objects.filter(is_active=True)
        return super().formfield_for_foreignkey(db_field, request, **kwargs)
```

## Resources

- [ModelAdmin Options](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#modeladmin-options) — Complete ModelAdmin reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*