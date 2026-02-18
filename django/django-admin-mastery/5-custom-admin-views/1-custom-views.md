---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-adding-custom-views"
---

# Adding Custom Admin Views

Django admin allows you to add custom views for reports, dashboards, and specialized functionality.

## Custom View on ModelAdmin

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

## Template for Custom View

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

## Adding Links to Custom Views

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

## Custom Admin Site Views

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

## Resources

- [ModelAdmin.get_urls()](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.get_urls) — Adding custom URLs to admin

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*