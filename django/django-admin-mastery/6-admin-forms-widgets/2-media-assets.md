---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-media-assets"
---

# Admin Media and JavaScript

## Introduction

Django admin supports adding custom CSS and JavaScript to enhance the interface with interactive features, custom styling, and dynamic behavior. This lesson covers the Media class, inline scripts, and best practices for admin asset management.

## Key Concepts

**Media class**: Inner class on ModelAdmin or Form that declares CSS and JS assets.

**django.jQuery**: The namespaced jQuery instance available in admin pages.

**extrastyle block**: Template block for adding extra CSS in admin templates.

**admin_change_form_document_ready**: Template block for inline JavaScript on the change form.

## Real World Context

A real estate listing admin needs a character counter on the description field, a map widget for the address field, and conditional field visibility based on property type. Adding these via the Media class keeps the admin enhancement code organized and loaded only on the pages that need it, without modifying Django core templates.

## Deep Dive

Add custom CSS and JavaScript to enhance admin functionality.

### Media Class on ModelAdmin

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    class Media:
        css = {
            'all': (
                'admin/css/custom_admin.css',
                'https://cdn.example.com/lib.css',
            )
        }
        js = (
            'admin/js/jquery.min.js',
            'admin/js/custom_admin.js',
        )
```

### Dynamic Media Based on View

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    def change_view(self, request, object_id, form_url='', extra_context=None):
        extra_context = extra_context or {}
        extra_context['extra_js'] = ['admin/js/article_editor.js']
        return super().change_view(
            request, object_id, form_url, extra_context
        )
```

### Custom JavaScript for Admin

```javascript
// static/admin/js/custom_admin.js
(function($) {
    'use strict';
    
    $(document).ready(function() {
        // Auto-generate slug from title
        $('#id_title').on('blur', function() {
            var title = $(this).val();
            var slug = title.toLowerCase()
                .replace(/[^a-z0-9]+/g, '-')
                .replace(/^-+|-+$/g, '');
            
            if (!$('#id_slug').val()) {
                $('#id_slug').val(slug);
            }
        });
        
        // Show/hide fields based on status
        function toggleScheduleFields() {
            var status = $('#id_status').val();
            if (status === 'scheduled') {
                $('.field-pub_date').show();
            } else {
                $('.field-pub_date').hide();
            }
        }
        
        $('#id_status').on('change', toggleScheduleFields);
        toggleScheduleFields();
        
        // Character counter
        var $textarea = $('#id_body');
        var $counter = $('<span class="char-counter"></span>');
        $textarea.after($counter);
        
        function updateCounter() {
            $counter.text($textarea.val().length + ' characters');
        }
        
        $textarea.on('input', updateCounter);
        updateCounter();
    });
})(django.jQuery);
```

### Custom CSS for Admin

```css
/* static/admin/css/custom_admin.css */

/* Custom card styles */
.dashboard-stats {
    display: flex;
    gap: 20px;
    margin-bottom: 30px;
}

.stat-card {
    background: #fff;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 20px;
    text-align: center;
    flex: 1;
}

.stat-card h3 {
    margin: 0 0 10px;
    color: #666;
    font-size: 14px;
}

.stat-number {
    font-size: 36px;
    font-weight: bold;
    color: #092e20;
}

/* Field highlighting */
.field-title input {
    border: 2px solid #44b78b;
}

.field-title input:focus {
    outline: none;
    box-shadow: 0 0 5px rgba(68, 183, 139, 0.5);
}

/* Custom button styles */
.button.preview-btn {
    background: #417690;
    color: white;
    padding: 5px 15px;
    border-radius: 4px;
}

/* Character counter */
.char-counter {
    display: block;
    text-align: right;
    color: #999;
    font-size: 12px;
    margin-top: 5px;
}
```

### Inline JavaScript

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    change_form_template = 'admin/blog/article/change_form.html'
```

```html
<!-- templates/admin/blog/article/change_form.html -->
{% extends "admin/change_form.html" %}

{% block admin_change_form_document_ready %}
{{ block.super }}
<script>
(function($) {
    // Confirmation before publishing
    $('input[name="_save"]').click(function(e) {
        if ($('#id_status').val() === 'published') {
            if (!confirm('Are you sure you want to publish this article?')) {
                e.preventDefault();
            }
        }
    });
})(django.jQuery);
</script>
{% endblock %}
```

## Common Pitfalls

1. **Using `$` instead of `django.jQuery`**: The admin loads jQuery in a namespaced variable. Using bare `$` will fail if another library is loaded. Always use `(function($) { ... })(django.jQuery)` to safely alias it.

2. **Loading large external libraries unconditionally**: Adding a charting library to every admin page via the Media class wastes bandwidth. Use change_view() with extra_context to load heavy assets only on specific pages.

3. **Not checking if elements exist before binding**: Admin pages are dynamic. JavaScript that assumes an element exists will throw errors on list views where that element is not present.

## Best Practices

1. **Use static files, not CDN links in production**: CDN links can go down or introduce version mismatches. Bundle assets in your static files for reliability.

2. **Scope CSS selectors to admin classes**: Use `.module`, `#content`, or Django's admin CSS class names to avoid styling conflicts with the rest of the admin interface.

3. **Use the document-ready template block**: For page-specific JavaScript, use the `admin_change_form_document_ready` block in a custom template rather than the Media class to ensure the DOM is ready.

## Summary

- The inner Media class on ModelAdmin declares CSS and JavaScript assets.
- Use `(function($) { ... })(django.jQuery)` for safe jQuery usage.
- Override change_view() to load assets conditionally on specific pages.
- Custom templates with extrastyle and document-ready blocks enable targeted enhancements.
- Scope CSS selectors and check element existence to avoid conflicts.

## Resources

- [Admin JavaScript](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#modeladmin-asset-definitions) — Adding CSS and JavaScript to admin

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*