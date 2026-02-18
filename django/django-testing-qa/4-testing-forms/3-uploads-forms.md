---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-testing-file-uploads-forms"
---

# Testing File Uploads in Forms

## Introduction

File upload testing requires creating file-like objects and using the correct encoding in requests.

## Key Concepts

**SimpleUploadedFile**: In-memory file for testing.

**multipart/form-data**: Required encoding for file uploads.

## Deep Dive

### Creating Test Files

```python
from django.core.files.uploadedfile import SimpleUploadedFile

class FileUploadTests(TestCase):
    def test_form_accepts_valid_image(self):
        image = SimpleUploadedFile(
            name='test.jpg',
            content=b'\x47\x49\x46\x38\x39\x61',  # GIF header
            content_type='image/jpeg'
        )
        
        form = PhotoForm(data={}, files={'image': image})
        self.assertTrue(form.is_valid())
    
    def test_form_rejects_non_image(self):
        file = SimpleUploadedFile(
            name='test.txt',
            content=b'text content',
            content_type='text/plain'
        )
        
        form = PhotoForm(data={}, files={'image': file})
        self.assertFalse(form.is_valid())
```

### Testing File Upload Views

```python
class FileUploadViewTests(TestCase):
    def test_upload_view(self):
        self.client.force_login(self.user)
        
        image = SimpleUploadedFile(
            'test.jpg',
            b'\x89PNG\r\n\x1a\n',  # PNG header
            content_type='image/png'
        )
        
        response = self.client.post(
            reverse('upload'),
            {'title': 'Test', 'image': image},
            format='multipart'
        )
        
        self.assertEqual(response.status_code, 302)
        self.assertTrue(Photo.objects.filter(title='Test').exists())
```

### Testing File Size Validation

```python
class FileSizeValidationTests(TestCase):
    def test_rejects_large_file(self):
        # Create file larger than limit
        large_content = b'x' * (5 * 1024 * 1024 + 1)  # 5MB+
        file = SimpleUploadedFile('large.pdf', large_content)
        
        form = DocumentForm(data={}, files={'document': file})
        self.assertFalse(form.is_valid())
        self.assertIn('document', form.errors)
```

## Best Practices

1. **Use real file headers**: Some validators check magic bytes.
2. **Clean up files**: Delete test files in tearDown.
3. **Use temporary storage**: Configure FILE_UPLOAD_TEMP_DIR in test settings.

## Summary

Use SimpleUploadedFile for creating test files. Include proper content types and file headers for realistic testing. Test both valid files and validation errors.

## Resources

- [File Uploads](https://docs.djangoproject.com/en/6.0/topics/http/file-uploads/) — Django file uploads documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*