---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-adding-custom-views"
---

# Adding Custom Admin Views

## Introduction

Django admin supports custom views for reports, dashboards, and specialized workflows. By overriding get_urls() on a ModelAdmin or AdminSite, you can add any view to the admin while inheriting its authentication and permission system.

## Key Concepts

**get_urls()**: Method that returns URL patterns for the admin. Override it to add custom routes.

**admin_view()**: Wrapper that enforces admin authentication and CSRF protection on custom views.

**TemplateResponse**: Response class that uses Django's template system for rendering.

**each_context()**: Provides standard admin context variables (site_header, etc.) to custom templates.

## Real World Context

A marketing team wants a dashboard inside the admin showing article publication statistics, top authors, and content gaps. Instead of building a separate analytics application, a custom statistics_view on ArticleAdmin renders a TemplateResponse with aggregated data, giving the team real-time insights without leaving the admin.

## Deep Dive

Django admin allows you to add custom views for reports, dashboards, and specialized functionality.

### Custom View on ModelAdmin

```python
from django.contrib import admin
from django.urls import path
from django.template.response import TemplateResponse
from django.shortcuts import get_object_or_404
from .models import Article


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'status', 'pub_date']
    
    def get_urls(self):
        urls = super().get_urls()
        custom_urls = [
            path(
                'statistics/',
                self.admin_site.admin_view(self.statistics_view),
                name='article_statistics',
            ),
            path(
                '<int:pk>/preview/',
                self.admin_site.admin_view(self.preview_view),
                name='article_preview',
            ),
        ]
        return custom_urls + urls
    
    def statistics_view(self, request):
        """Custom statistics page."""
        context = {
            **self.admin_site.each_context(request),
            'title': 'Article Statistics',
            'total_articles': Article.objects.count(),
            'published': Article.objects.filter(status='published').count(),
            'drafts': Article.objects.filter(status='draft').count(),
            'by_author': Article.objects.values('author__username').annotate(
                count=Count('id')
            ).order_by('-count')[:10],
        }
        return TemplateResponse(
            request,
            'admin/blog/article/statistics.html',
            context
        )
    
    def preview_view(self, request, pk):
        """Preview an article."""
        article = get_object_or_404(Article, pk=pk)
        context = {
            **self.admin_site.each_context(request),
            'title': f'Preview: {article.title}',
            'article': article,
            'opts': self.model._meta,
        }
        return TemplateResponse(
            request,
            'admin/blog/article/preview.html',
            context
        )
```

### Template for Custom View

```html
<!-- templates/admin/blog/article/statistics.html -->
{% extends "admin/base_site.html" %}
{% load static %}

{% block content %}
<h1>{{ title }}</h1>

<div class="dashboard-stats">
    <div class="stat-card">
        <h3>Total Articles</h3>
        <span class="stat-number">{{ total_articles }}</span>
    </div>
    <div class="stat-card">
        <h3>Published</h3>
        <span class="stat-number">{{ published }}</span>
    </div>
    <div class="stat-card">
        <h3>Drafts</h3>
        <span class="stat-number">{{ drafts }}</span>
    </div>
</div>

<h2>Top Authors</h2>
<table>
    <thead>
        <tr>
            <th>Author</th>
            <th>Articles</th>
        </tr>
    </thead>
    <tbody>
        {% for author in by_author %}
        <tr>
            <td>{{ author.author__username }}</td>
            <td>{{ author.count }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

### Adding Links to Custom Views

```python
from django.utils.html import format_html
from django.urls import reverse

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'status', 'view_actions']
    change_list_template = 'admin/blog/article/change_list.html'
    
    @admin.display(description='Actions')
    def view_actions(self, obj):
        preview_url = reverse('admin:article_preview', args=[obj.pk])
        return format_html(
            '<a href="{}" class="button">Preview</a>',
            preview_url
        )
```

```html
<!-- templates/admin/blog/article/change_list.html -->
{% extends "admin/change_list.html" %}

{% block object-tools-items %}
<li>
    <a href="{% url 'admin:article_statistics' %}" class="viewlink">
        View Statistics
    </a>
</li>
{{ block.super }}
{% endblock %}
```

### Custom Admin Site Views

```python
from django.contrib.admin import AdminSite


class MyAdminSite(AdminSite):
    def get_urls(self):
        urls = super().get_urls()
        custom_urls = [
            path(
                'dashboard/',
                self.admin_view(self.dashboard_view),
                name='dashboard',
            ),
            path(
                'reports/',
                self.admin_view(self.reports_view),
                name='reports',
            ),
        ]
        return custom_urls + urls
    
    def dashboard_view(self, request):
        from .models import Article, User, Order
        
        context = {
            **self.each_context(request),
            'title': 'Dashboard',
            'recent_articles': Article.objects.order_by('-created_at')[:5],
            'new_users': User.objects.filter(
                date_joined__gte=timezone.now() - timedelta(days=7)
            ).count(),
            'pending_orders': Order.objects.filter(status='pending').count(),
        }
        return TemplateResponse(request, 'admin/dashboard.html', context)
```

## Common Pitfalls

1. **Placing custom URLs after super().get_urls()**: Django's default URL patterns include catch-all patterns. If your custom URLs come after them, they will never match. Always prepend custom URLs.

2. **Forgetting to wrap views with admin_view()**: Without self.admin_site.admin_view(), your custom view has no authentication check, making it publicly accessible to anyone who knows the URL.

3. **Not including each_context()**: Custom views that skip `self.admin_site.each_context(request)` in the template context will render without the standard admin header, sidebar, and navigation.

## Best Practices

1. **Use TemplateResponse instead of render()**: TemplateResponse allows middleware to modify the response before rendering, which is the pattern Django admin uses internally.

2. **Link custom views from the change list**: Add buttons in the object-tools block or use format_html in list_display to make custom views discoverable.

3. **Reuse admin base templates**: Extend admin/base_site.html so your custom pages get the admin look and feel, including breadcrumbs and navigation.

## Summary

- Override get_urls() to add custom URL patterns to any ModelAdmin or AdminSite.
- Wrap custom views with admin_site.admin_view() for authentication.
- Include each_context() in template context for the standard admin chrome.
- Place custom URLs before super().get_urls() to avoid catch-all conflicts.
- Use TemplateResponse and extend admin templates for consistent styling.

## Code Examples

**Adding a custom statistics view to ModelAdmin via get_urls()**

```python
class MyModelAdmin(admin.ModelAdmin):
    def get_urls(self):
        urls = super().get_urls()
        custom_urls = [
            path('statistics/',
                 self.admin_site.admin_view(self.statistics_view),
                 name='model_statistics'),
        ]
        return custom_urls + urls

    def statistics_view(self, request):
        context = {
            **self.admin_site.each_context(request),
            'title': 'Statistics',
        }
        return TemplateResponse(request, 'admin/statistics.html', context)
```


## Resources

- [ModelAdmin.get_urls()](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.get_urls) — Adding custom URLs to admin

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*