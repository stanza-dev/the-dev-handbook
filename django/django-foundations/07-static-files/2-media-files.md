---
source_course: "django-foundations"
source_lesson: "django-foundations-media-files"
---

# Handling User Uploads

Media files are user-uploaded content: profile pictures, documents, attachments. Unlike static files, these are dynamic and stored separately.

## Configuring Media Files

First, define where uploaded files will be stored and what URL prefix will serve them.

```python
# mysite/settings.py

# URL prefix for media files
MEDIA_URL = 'media/'

# Directory for uploaded files
MEDIA_ROOT = BASE_DIR / 'media'
```

## Serving Media in Development

Add to your URL configuration:

```python
# mysite/urls.py
from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
]

# Only in development!
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Adding File Fields to Models

Use `ImageField` for images and `FileField` for other file types. The `upload_to` argument determines the subdirectory inside `MEDIA_ROOT` where files are saved.

```python
# polls/models.py
from django.db import models


class UserProfile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    
    # Image field with upload directory
    avatar = models.ImageField(
        upload_to='avatars/',
        blank=True,
        null=True
    )
    
    # File field for documents
    resume = models.FileField(
        upload_to='resumes/',
        blank=True
    )
    
    # Dynamic upload path
    def user_directory_path(instance, filename):
        return f'user_{instance.user.id}/{filename}'
    
    document = models.FileField(upload_to=user_directory_path)
```

**Note**: For `ImageField`, install Pillow:

```bash
pip install Pillow
```

## Creating Upload Forms

Create a ModelForm that includes the file fields from your model.

```python
# polls/forms.py
from django import forms
from .models import UserProfile


class ProfileForm(forms.ModelForm):
    class Meta:
        model = UserProfile
        fields = ['avatar', 'resume']
```

## Handling Uploads in Views

When processing file uploads, you must pass both `request.POST` and `request.FILES` to the form constructor.

```python
# polls/views.py
from django.shortcuts import render, redirect
from .forms import ProfileForm


def upload_profile(request):
    if request.method == 'POST':
        form = ProfileForm(
            request.POST,
            request.FILES,  # Important! Include files
            instance=request.user.userprofile
        )
        if form.is_valid():
            form.save()
            return redirect('profile')
    else:
        form = ProfileForm(instance=request.user.userprofile)
    
    return render(request, 'upload.html', {'form': form})
```

## Template for File Uploads

The form template must include `enctype="multipart/form-data"` to support file uploads. Without it, the browser sends only text data.

```html
<!-- templates/upload.html -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Upload</button>
</form>

<!-- Display uploaded image -->
{% if profile.avatar %}
    <img src="{{ profile.avatar.url }}" alt="Avatar">
{% endif %}
```

**Important**: Always use `enctype="multipart/form-data"` for file uploads!

## File Field Methods

Once a file is uploaded, the field provides several useful attributes for accessing the file's URL, path, name, and size.

```python
# In views or templates
profile = UserProfile.objects.get(pk=1)

# File URL (for templates)
profile.avatar.url      # '/media/avatars/photo.jpg'

# File path (on disk)
profile.avatar.path     # '/path/to/media/avatars/photo.jpg'

# File name
profile.avatar.name     # 'avatars/photo.jpg'

# File size (bytes)
profile.avatar.size     # 102400

# Check if file exists
if profile.avatar:
    print("Has avatar")
```

## Common Pitfalls

- **Forgetting `enctype="multipart/form-data"`**: Without this form attribute, `request.FILES` will be empty and file uploads silently fail.
- **Not passing `request.FILES` to the form**: When processing file uploads, you must pass both `request.POST` and `request.FILES` to the form constructor.
- **Serving media files without DEBUG check**: Only add the media URL pattern inside `if settings.DEBUG:` to avoid serving user uploads through Django in production.

## Best Practices

- **Install Pillow for ImageField**: `ImageField` requires the Pillow library (`pip install Pillow`) for image validation.
- **Use dynamic `upload_to` paths**: Define a callable for `upload_to` to organize files by user or date (e.g., `user_{id}/filename`).
- **Validate file types and sizes**: Add custom validation to reject oversized files or disallowed file types.

## Summary

- Media files are **user-uploaded content** stored separately from static files
- Configure `MEDIA_URL` and `MEDIA_ROOT` in settings for upload destinations
- Use `ImageField` and `FileField` in models with `upload_to` for directory organization
- Forms must use `enctype="multipart/form-data"` and views must pass `request.FILES`
- Access uploaded files via `.url`, `.path`, `.name`, and `.size` attributes on the field

## Code Examples

**Model with file and image upload fields, plus the required form configuration for uploads**

```python
from django.db import models

class UserProfile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    resume = models.FileField(upload_to='resumes/', blank=True)

# In views: form = ProfileForm(request.POST, request.FILES)
# In templates: <form method="post" enctype="multipart/form-data">
```


## Resources

- [Managing Files](https://docs.djangoproject.com/en/6.0/topics/files/) — Official guide to file handling in Django

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*