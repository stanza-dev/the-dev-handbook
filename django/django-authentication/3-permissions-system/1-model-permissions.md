---
source_course: "django-authentication"
source_lesson: "django-authentication-model-permissions"
---

# Model Permissions

Django automatically creates permissions for each model. Learn to use and customize them.

## Default Model Permissions

For each model, Django creates:
- `add_<model>` - Can add instances
- `change_<model>` - Can edit instances
- `delete_<model>` - Can delete instances
- `view_<model>` - Can view instances

```python
# For model Article in app 'blog'
# Permissions created:
# - blog.add_article
# - blog.change_article
# - blog.delete_article
# - blog.view_article
```

## Checking Permissions

```python
# Check if user has permission
user.has_perm('blog.add_article')       # Single permission
user.has_perm('blog.change_article')

# Check multiple permissions (all required)
user.has_perms(['blog.add_article', 'blog.change_article'])

# Get all permissions
user.get_all_permissions()
# {'blog.add_article', 'blog.change_article', ...}

# Get group permissions
user.get_group_permissions()
```

## In Views

```python
from django.contrib.auth.decorators import permission_required

# Single permission
@permission_required('blog.add_article')
def create_article(request):
    pass

# Multiple permissions (all required)
@permission_required(['blog.add_article', 'blog.change_article'])
def manage_article(request):
    pass

# Raise 403 instead of redirecting to login
@permission_required('blog.delete_article', raise_exception=True)
def delete_article(request, pk):
    pass

# Manual check in view
def article_view(request):
    if not request.user.has_perm('blog.view_article'):
        return HttpResponseForbidden("Permission denied")
    # ...
```

## In Templates

```html
{% if perms.blog.add_article %}
    <a href="{% url 'article_create' %}">Create Article</a>
{% endif %}

{% if perms.blog.change_article %}
    <a href="{% url 'article_edit' article.pk %}">Edit</a>
{% endif %}

{% if perms.blog %}
    <!-- User has ANY permission in blog app -->
{% endif %}
```

## Custom Model Permissions

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    is_published = models.BooleanField(default=False)
    
    class Meta:
        permissions = [
            ('publish_article', 'Can publish articles'),
            ('feature_article', 'Can feature articles on homepage'),
            ('archive_article', 'Can archive articles'),
        ]

# After migration:
# - blog.publish_article
# - blog.feature_article
# - blog.archive_article
```

## Assigning Permissions

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

# Get permission
permission = Permission.objects.get(
    codename='publish_article',
    content_type__app_label='blog'
)

# Assign to user
user.user_permissions.add(permission)

# Remove from user
user.user_permissions.remove(permission)

# Clear all permissions
user.user_permissions.clear()

# Check (must refetch user or clear permission cache)
user = User.objects.get(pk=user.pk)  # Refetch
user.has_perm('blog.publish_article')
```

## Creating Permissions Programmatically

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(Article)

permission, created = Permission.objects.get_or_create(
    codename='export_article',
    name='Can export articles',
    content_type=content_type
)
```

## Customizing Default Permissions

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    
    class Meta:
        # Only create add and view permissions
        default_permissions = ('add', 'view')
        
        # Additional custom permissions
        permissions = [
            ('publish_article', 'Can publish'),
        ]
```

## Resources

- [Permissions and Authorization](https://docs.djangoproject.com/en/6.0/topics/auth/default/#permissions-and-authorization) — Official permissions documentation

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*