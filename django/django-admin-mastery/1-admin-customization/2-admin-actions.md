---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-actions"
---

# Custom Admin Actions

## Introduction

Admin actions let you perform bulk operations on selected objects directly from the change list. In this lesson, you will learn how to create custom actions for publishing, exporting, and managing records in bulk.

## Key Concepts

**Admin action**: A function that receives the ModelAdmin, the HTTP request, and a queryset of selected objects.

**@admin.action**: Decorator that sets the human-readable description shown in the action dropdown.

**message_user()**: Method to display success or error feedback after an action completes.

**get_actions()**: Method to dynamically add or remove actions based on user permissions.

## Real World Context

In an e-commerce back office, a customer support team receives dozens of refund requests daily. Without bulk actions, each order must be opened individually to change its status. A custom 'Mark as refunded' action lets the team select 20 orders at once from the change list and process them in a single click, saving hours of repetitive work.

## Deep Dive

Admin actions let users perform bulk operations on selected objects.

### Basic Action

```python
from django.contrib import admin
from django.contrib import messages


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'status', 'pub_date']
    actions = ['make_published', 'make_draft', 'export_as_csv']
    
    @admin.action(description='Mark selected articles as published')
    def make_published(self, request, queryset):
        count = queryset.update(status='published')
        self.message_user(
            request,
            f'{count} articles marked as published.',
            messages.SUCCESS
        )
    
    @admin.action(description='Mark selected articles as draft')
    def make_draft(self, request, queryset):
        count = queryset.update(status='draft')
        self.message_user(request, f'{count} articles marked as draft.')
```

### Export Action

```python
import csv
from django.http import HttpResponse

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['export_as_csv']
    
    @admin.action(description='Export selected as CSV')
    def export_as_csv(self, request, queryset):
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="articles.csv"'
        
        writer = csv.writer(response)
        writer.writerow(['Title', 'Author', 'Status', 'Published'])  # Header
        
        for article in queryset:
            writer.writerow([
                article.title,
                article.author.username,
                article.status,
                article.pub_date,
            ])
        
        return response
```

### Action with Confirmation

```python
from django.contrib.admin import helpers
from django.template.response import TemplateResponse

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    actions = ['delete_with_confirmation']
    
    @admin.action(description='Delete with custom confirmation')
    def delete_with_confirmation(self, request, queryset):
        if 'apply' in request.POST:
            # Perform the action
            count = queryset.count()
            queryset.delete()
            self.message_user(request, f'Deleted {count} articles.')
            return None
        
        # Show confirmation page
        context = {
            'title': 'Confirm Deletion',
            'queryset': queryset,
            'action_checkbox_name': helpers.ACTION_CHECKBOX_NAME,
            'opts': self.model._meta,
        }
        return TemplateResponse(
            request,
            'admin/confirm_delete.html',
            context
        )
```

### Conditional Actions

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def get_actions(self, request):
        actions = super().get_actions(request)
        
        # Remove delete action for non-superusers
        if not request.user.is_superuser:
            if 'delete_selected' in actions:
                del actions['delete_selected']
        
        return actions
    
    @admin.action(description='Send notification to authors')
    def notify_authors(self, request, queryset):
        if not request.user.has_perm('blog.send_notifications'):
            self.message_user(
                request,
                'You do not have permission to send notifications.',
                messages.ERROR
            )
            return
        
        # Send notifications...
        for article in queryset:
            send_notification(article.author, f'Update on: {article.title}')
        
        self.message_user(
            request,
            f'Notifications sent to {queryset.count()} authors.'
        )
```

### Global Actions

```python
# admin.py
from django.contrib import admin


def export_selected_objects(modeladmin, request, queryset):
    """Generic export action for any model."""
    import csv
    from django.http import HttpResponse
    
    meta = modeladmin.model._meta
    field_names = [field.name for field in meta.fields]
    
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = f'attachment; filename="{meta}.csv"'
    
    writer = csv.writer(response)
    writer.writerow(field_names)
    
    for obj in queryset:
        writer.writerow([getattr(obj, field) for field in field_names])
    
    return response

export_selected_objects.short_description = 'Export selected objects'

# Add to all admin sites
admin.site.add_action(export_selected_objects, 'export_selected')
```

## Common Pitfalls

1. **Forgetting to call message_user()**: Actions that complete silently leave admins wondering if anything happened. Always provide feedback, even on partial success.

2. **Not handling empty querysets**: If the user selects no objects or the queryset returns zero matches, your action should detect this and display a warning instead of silently doing nothing.

3. **Using queryset.delete() without confirmation**: Destructive actions should either use Django's built-in deletion confirmation or implement a custom confirmation step to prevent accidental data loss.

## Best Practices

1. **Use @admin.action(permissions=[...])** to restrict actions to users with specific permissions rather than checking permissions inside the function body.

2. **Return an HttpResponse for exports**: Actions that generate files (CSV, PDF) should return an HttpResponse with the appropriate content type and Content-Disposition header.

3. **Keep actions focused**: Each action should do one thing. Instead of a single action with multiple behaviors based on flags, create separate named actions.

## Summary

- Admin actions process selected objects in bulk from the change list.
- Define actions as methods on ModelAdmin and list them in the actions attribute.
- The @admin.action decorator sets the display name and optional permissions.
- Always call message_user() to provide feedback after the action completes.
- Use get_actions() to conditionally show or hide actions based on user permissions.

## Resources

- [Admin Actions](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/actions/) — Official admin actions documentation

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*