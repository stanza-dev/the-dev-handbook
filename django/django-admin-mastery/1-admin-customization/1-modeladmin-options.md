---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-modeladmin-options"
---

# ModelAdmin Configuration

## Introduction

The ModelAdmin class is the backbone of Django's admin customization. In this lesson, you will learn how to configure list displays, search, filtering, and form layouts to build an efficient admin interface tailored to your models.

## Key Concepts

**ModelAdmin**: The class that controls how a model is displayed and edited in the admin.

**list_display**: A tuple or list of field names to show as columns in the change list.

**list_filter**: Fields that generate sidebar filters on the change list.

**search_fields**: Fields searched when using the admin search box.

**fieldsets**: Groups of fields displayed together on the change form.

**prepopulated_fields**: Fields auto-filled from other fields, commonly used for slugs.

## Real World Context

Imagine you are building a content management system for a news site. Editors need to quickly scan hundreds of articles, filter by status or author, and search by title. Without proper ModelAdmin configuration, every model shows only its __str__ in a single column with no filtering or search, forcing editors to click into each record. Proper list_display, list_filter, and search_fields turn the admin into a productive editorial dashboard.

## Deep Dive

The ModelAdmin class provides extensive options for customizing how models appear in the admin interface.

### Basic ModelAdmin

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

### Display Options

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

### Change Form Configuration

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

### Permission Control

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

### Filtering Querysets

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

## Common Pitfalls

1. **Putting a field in both list_display_links and list_editable**: Django requires that the field linking to the change page cannot be editable inline. If you need inline editing, set list_display_links to a different field first.

2. **Using search_fields on non-indexed columns**: Searching large tables with icontains on unindexed text columns causes slow queries. Add database indexes or use the `^` prefix for startswith matching.

3. **Forgetting to include ForeignKey traversals in search_fields**: Writing `search_fields = ['author']` searches the FK id. You need `author__username` to search the related model's field.

## Best Practices

1. **Always set list_display**: The default single-column view is nearly useless. Show 4-6 key fields that help users identify records at a glance.

2. **Use date_hierarchy for time-series models**: It adds a convenient date drill-down navigation bar above the list and costs nothing to implement.

3. **Set list_per_page to a reasonable number**: The default 100 can be slow for complex querysets. 25-50 is usually a better choice for admin usability.

## Summary

- ModelAdmin controls every aspect of how a model appears in the admin.
- list_display, list_filter, and search_fields make the change list usable.
- fieldsets organize the change form into logical sections.
- prepopulated_fields and raw_id_fields improve the editing experience.
- Permission methods (has_add_permission, etc.) control per-object access.

## Code Examples

**Basic ModelAdmin configuration with common list display, filter, and search options**

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'status', 'pub_date']
    list_filter = ['status', 'pub_date', 'author']
    search_fields = ['title', 'body']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'pub_date'
    ordering = ['-pub_date']
```


## Resources

- [ModelAdmin Options](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#modeladmin-options) — Complete ModelAdmin reference

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*