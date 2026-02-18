---
source_course: "django-foundations"
source_lesson: "django-foundations-media-files"
---

# Handling User Uploads

Media files are user-uploaded content: profile pictures, documents, attachments. Unlike static files, these are dynamic and stored separately.

## Configuring Media Files

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

## Resources

- [Managing Files](https://docs.djangoproject.com/en/6.0/topics/files/) — Official guide to file handling in Django

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*