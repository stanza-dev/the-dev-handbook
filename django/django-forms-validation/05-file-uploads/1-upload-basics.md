---
source_course: "django-forms-validation"
source_lesson: "django-forms-validation-file-upload-basics"
---

# File Upload Basics

Handling file uploads requires proper form encoding, validation, and storage configuration.

## Form Configuration

```python
from django import forms


class UploadForm(forms.Form):
    title = forms.CharField(max_length=100)
    document = forms.FileField()
    image = forms.ImageField(required=False)  # Requires Pillow
```

## Template Requirements

```html
<!-- enctype is required for file uploads! -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Upload</button>
</form>
```

## View Handling

```python
def upload_file(request):
    if request.method == 'POST':
        form = UploadForm(request.POST, request.FILES)  # Include FILES!
        if form.is_valid():
            handle_uploaded_file(request.FILES['document'])
            return redirect('success')
    else:
        form = UploadForm()
    
    return render(request, 'upload.html', {'form': form})


def handle_uploaded_file(f):
    """Save uploaded file to disk."""
    with open(f'uploads/{f.name}', 'wb+') as destination:
        for chunk in f.chunks():
            destination.write(chunk)
```

## Using FileField on Models

```python
# models.py
from django.db import models


class Document(models.Model):
    title = forms.CharField(max_length=100)
    file = models.FileField(upload_to='documents/')
    uploaded_at = models.DateTimeField(auto_now_add=True)


class Photo(models.Model):
    title = forms.CharField(max_length=100)
    image = models.ImageField(upload_to='photos/%Y/%m/')  # Organized by date
    thumbnail = models.ImageField(upload_to='thumbs/', blank=True)
```

## ModelForm with Files

```python
class DocumentForm(forms.ModelForm):
    class Meta:
        model = Document
        fields = ['title', 'file']


def upload_document(request):
    if request.method == 'POST':
        form = DocumentForm(request.POST, request.FILES)
        if form.is_valid():
            document = form.save()
            return redirect('document_detail', pk=document.pk)
    else:
        form = DocumentForm()
    
    return render(request, 'upload.html', {'form': form})
```

## File Validation

```python
from django.core.validators import FileExtensionValidator
from django.core.exceptions import ValidationError


def validate_file_size(value):
    filesize = value.size
    if filesize > 10 * 1024 * 1024:  # 10 MB
        raise ValidationError('Maximum file size is 10MB')


class DocumentForm(forms.Form):
    file = forms.FileField(
        validators=[
            FileExtensionValidator(allowed_extensions=['pdf', 'doc', 'docx']),
            validate_file_size,
        ]
    )
```

## Custom File Validation

```python
class ImageUploadForm(forms.Form):
    image = forms.ImageField()
    
    def clean_image(self):
        image = self.cleaned_data['image']
        
        # Check file size
        if image.size > 5 * 1024 * 1024:  # 5 MB
            raise ValidationError('Image must be smaller than 5MB')
        
        # Check dimensions (requires Pillow)
        from PIL import Image
        img = Image.open(image)
        
        if img.width > 4000 or img.height > 4000:
            raise ValidationError('Image dimensions too large')
        
        if img.width < 100 or img.height < 100:
            raise ValidationError('Image too small')
        
        # Reset file position
        image.seek(0)
        
        return image
```

## Settings Configuration

```python
# settings.py
import os

# Media files (user uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Maximum upload size (default is 2.5MB)
DATA_UPLOAD_MAX_MEMORY_SIZE = 10 * 1024 * 1024  # 10 MB
FILE_UPLOAD_MAX_MEMORY_SIZE = 10 * 1024 * 1024  # 10 MB
```

```python
# urls.py (development only)
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... your urls
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Resources

- [File Uploads](https://docs.djangoproject.com/en/6.0/topics/http/file-uploads/) — Official file uploads documentation

---

> 📘 *This lesson is part of the [Django Forms & Validation](https://stanza.dev/courses/django-forms-validation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*