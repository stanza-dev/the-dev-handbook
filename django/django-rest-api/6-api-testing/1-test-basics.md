---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-test-basics"
---

# API Testing Fundamentals

Testing APIs is essential for reliability. Django's test client makes it easy to test your endpoints.

## Basic API Test

```python
import json
from django.test import TestCase, Client
from django.contrib.auth.models import User
from .models import Article


class ArticleAPITests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass'
        )
        self.article = Article.objects.create(
            title='Test Article',
            body='Test content',
            author=self.user
        )
    
    def test_list_articles(self):
        """Test GET /api/articles/ returns list."""
        response = self.client.get('/api/articles/')
        
        self.assertEqual(response.status_code, 200)
        
        data = response.json()
        self.assertIn('articles', data)
        self.assertEqual(len(data['articles']), 1)
        self.assertEqual(data['articles'][0]['title'], 'Test Article')
    
    def test_get_article_detail(self):
        """Test GET /api/articles/{id}/ returns single article."""
        response = self.client.get(f'/api/articles/{self.article.id}/')
        
        self.assertEqual(response.status_code, 200)
        
        data = response.json()
        self.assertEqual(data['id'], self.article.id)
        self.assertEqual(data['title'], 'Test Article')
    
    def test_get_nonexistent_article(self):
        """Test GET /api/articles/{id}/ returns 404 for missing article."""
        response = self.client.get('/api/articles/99999/')
        
        self.assertEqual(response.status_code, 404)
```

## Testing POST Requests

```python
class ArticleCreateTests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass'
        )
    
    def test_create_article(self):
        """Test POST /api/articles/ creates an article."""
        self.client.login(username='testuser', password='testpass')
        
        payload = {
            'title': 'New Article',
            'body': 'Article content'
        }
        
        response = self.client.post(
            '/api/articles/',
            data=json.dumps(payload),
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, 201)
        
        data = response.json()
        self.assertIn('id', data)
        
        # Verify in database
        article = Article.objects.get(id=data['id'])
        self.assertEqual(article.title, 'New Article')
    
    def test_create_article_validation_error(self):
        """Test POST /api/articles/ returns 400 for invalid data."""
        self.client.login(username='testuser', password='testpass')
        
        # Missing required field
        payload = {'body': 'No title'}
        
        response = self.client.post(
            '/api/articles/',
            data=json.dumps(payload),
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, 400)
        
        data = response.json()
        self.assertIn('error', data)
    
    def test_create_article_unauthenticated(self):
        """Test POST /api/articles/ requires authentication."""
        payload = {'title': 'Test', 'body': 'Content'}
        
        response = self.client.post(
            '/api/articles/',
            data=json.dumps(payload),
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, 401)
```

## Testing with Token Authentication

```python
from .models import APIToken


class TokenAuthTests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass'
        )
        self.token = APIToken.objects.create(user=self.user)
    
    def test_authenticated_request(self):
        """Test request with valid token succeeds."""
        response = self.client.get(
            '/api/profile/',
            HTTP_AUTHORIZATION=f'Bearer {self.token.key}'
        )
        
        self.assertEqual(response.status_code, 200)
    
    def test_invalid_token(self):
        """Test request with invalid token fails."""
        response = self.client.get(
            '/api/profile/',
            HTTP_AUTHORIZATION='Bearer invalidtoken'
        )
        
        self.assertEqual(response.status_code, 401)
    
    def test_login_returns_token(self):
        """Test login endpoint returns token."""
        payload = {
            'username': 'testuser',
            'password': 'testpass'
        }
        
        response = self.client.post(
            '/api/login/',
            data=json.dumps(payload),
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, 200)
        
        data = response.json()
        self.assertIn('token', data)
        self.assertTrue(len(data['token']) > 0)
```

## Test Helper Methods

```python
class APITestCase(TestCase):
    """Base test case with helper methods."""
    
    def api_client(self, user=None):
        """Get authenticated API client."""
        client = Client()
        if user:
            token = APIToken.objects.create(user=user)
            client.defaults['HTTP_AUTHORIZATION'] = f'Bearer {token.key}'
        return client
    
    def assertAPISuccess(self, response):
        """Assert API response is successful."""
        self.assertIn(response.status_code, [200, 201, 204])
    
    def assertAPIError(self, response, status_code, error_code=None):
        """Assert API response is an error."""
        self.assertEqual(response.status_code, status_code)
        if error_code:
            data = response.json()
            self.assertEqual(data['error']['code'], error_code)


class ArticleAPITests(APITestCase):
    def test_create_article(self):
        user = User.objects.create_user('test', 'test@test.com', 'pass')
        client = self.api_client(user)
        
        response = client.post(
            '/api/articles/',
            data=json.dumps({'title': 'Test', 'body': 'Content'}),
            content_type='application/json'
        )
        
        self.assertAPISuccess(response)
```

## Resources

- [Testing in Django](https://docs.djangoproject.com/en/6.0/topics/testing/) — Official Django testing documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*