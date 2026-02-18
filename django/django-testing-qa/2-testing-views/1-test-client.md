---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-test-client"
---

# The Django Test Client

The test client simulates a browser, allowing you to test views without running a server.

## Basic Usage

```python
from django.test import TestCase, Client
from django.urls import reverse


class ViewTests(TestCase):
    def setUp(self):
        self.client = Client()
    
    def test_homepage(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
    
    def test_article_list(self):
        response = self.client.get(reverse('article-list'))
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'blog/article_list.html')
    
    def test_article_detail(self):
        article = Article.objects.create(title='Test', slug='test')
        response = self.client.get(
            reverse('article-detail', kwargs={'slug': 'test'})
        )
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Test')
```

## Response Assertions

```python
class ResponseTests(TestCase):
    def test_response_assertions(self):
        response = self.client.get('/articles/')
        
        # Status code
        self.assertEqual(response.status_code, 200)
        
        # Content contains text
        self.assertContains(response, 'Articles')
        self.assertNotContains(response, 'Error')
        
        # Template used
        self.assertTemplateUsed(response, 'blog/article_list.html')
        self.assertTemplateNotUsed(response, 'blog/error.html')
        
        # Context data
        self.assertIn('articles', response.context)
        self.assertEqual(len(response.context['articles']), 5)
        
        # Redirect
        response = self.client.get('/old-url/')
        self.assertRedirects(response, '/new-url/')
        
        # JSON response
        response = self.client.get('/api/articles/')
        self.assertEqual(response['Content-Type'], 'application/json')
        data = response.json()
        self.assertIsInstance(data, list)
```

## POST Requests

```python
class FormTests(TestCase):
    def test_create_article(self):
        user = User.objects.create_user('test', 'test@test.com', 'pass')
        self.client.login(username='test', password='pass')
        
        response = self.client.post(
            reverse('article-create'),
            data={
                'title': 'New Article',
                'body': 'Article content',
                'status': 'draft',
            }
        )
        
        # Check redirect after successful creation
        self.assertEqual(response.status_code, 302)
        
        # Verify article was created
        self.assertTrue(Article.objects.filter(title='New Article').exists())
    
    def test_invalid_form(self):
        self.client.login(username='test', password='pass')
        
        response = self.client.post(
            reverse('article-create'),
            data={'title': ''}  # Missing required fields
        )
        
        # Form errors, stays on same page
        self.assertEqual(response.status_code, 200)
        self.assertFormError(response, 'form', 'title', 'This field is required.')
```

## Authentication in Tests

```python
class AuthenticatedViewTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
    
    def test_login_required_view(self):
        # Unauthenticated request
        response = self.client.get(reverse('dashboard'))
        self.assertRedirects(
            response,
            f'/accounts/login/?next={reverse("dashboard")}'
        )
    
    def test_authenticated_access(self):
        # Login
        self.client.login(username='testuser', password='testpass123')
        
        response = self.client.get(reverse('dashboard'))
        self.assertEqual(response.status_code, 200)
    
    def test_force_login(self):
        # Force login without password (faster)
        self.client.force_login(self.user)
        
        response = self.client.get(reverse('dashboard'))
        self.assertEqual(response.status_code, 200)
    
    def test_logout(self):
        self.client.force_login(self.user)
        self.client.logout()
        
        response = self.client.get(reverse('dashboard'))
        self.assertEqual(response.status_code, 302)  # Redirects to login
```

## File Uploads

```python
from django.core.files.uploadedfile import SimpleUploadedFile


class FileUploadTests(TestCase):
    def test_image_upload(self):
        self.client.force_login(self.user)
        
        # Create a test image
        image = SimpleUploadedFile(
            name='test_image.jpg',
            content=b'\x47\x49\x46\x38\x39\x61\x01\x00\x01\x00',  # Minimal GIF
            content_type='image/jpeg'
        )
        
        response = self.client.post(
            reverse('profile-update'),
            data={
                'name': 'Test User',
                'avatar': image,
            }
        )
        
        self.assertEqual(response.status_code, 302)
        profile = Profile.objects.get(user=self.user)
        self.assertTrue(profile.avatar.name.endswith('.jpg'))
```

## Resources

- [Test Client](https://docs.djangoproject.com/en/6.0/topics/testing/tools/#the-test-client) — Django test client documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*