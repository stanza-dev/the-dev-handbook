---
source_course: "django-admin-mastery"
source_lesson: "django-admin-mastery-admin-media-assets"
---

# Admin Media and JavaScript

Add custom CSS and JavaScript to enhance admin functionality.

## Media Class on ModelAdmin

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

## Dynamic Media Based on View

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

## Custom JavaScript for Admin

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

## Custom CSS for Admin

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

## Inline JavaScript

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

## Resources

- [Admin JavaScript](https://docs.djangoproject.com/en/6.0/ref/contrib/admin/#modeladmin-asset-definitions) — Adding CSS and JavaScript to admin

---

> 📘 *This lesson is part of the [Django Admin Mastery](https://stanza.dev/courses/django-admin-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*