---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-test-basics"
---

# API Testing Fundamentals

## Introduction

Testing APIs is essential for reliability. Without automated tests, every code change risks breaking existing functionality. Django's test framework provides powerful tools for testing API endpoints with minimal setup.

Testing APIs is essential for reliability. Django's test client makes it easy to test your endpoints.


## Key Concepts

**Test Client**: Django's built-in `Client` class that simulates HTTP requests without a running server.

**TestCase**: Django's test class that wraps each test in a database transaction for isolation.

**Response Assertions**: Methods like `assertEqual(response.status_code, 200)` and `response.json()` for verifying API behavior.

**Test Isolation**: Each test runs in its own transaction, so database changes from one test don't affect others.


## Real World Context

API testing is critical for:
- **Continuous integration**: Automated tests run on every commit to catch regressions
- **Refactoring safety**: Tests give you confidence to restructure code without breaking functionality
- **Documentation**: Tests serve as executable examples of how the API works
- **Contract verification**: Tests ensure the API matches its documented specification

## Basic API Test

Django's test `Client` simulates HTTP requests without a running server. Each test method verifies one specific behavior:

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

The `setUp()` method creates fresh test data before each test. Using `response.json()` parses the JSON body and returns a Python dictionary for easy assertions.

## Testing POST Requests

POST tests should verify successful creation, validation errors, and authentication requirements:

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
            data=payload,
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
            data=payload,
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
            data=payload,
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, 401)
```

Notice the `content_type='application/json'` parameter -- without it, Django treats the data as form-encoded. The assertion checks both the status code and the database to confirm the article was actually created.

## Testing with Token Authentication

For token-authenticated endpoints, create a token in `setUp()` and pass it via the `HTTP_AUTHORIZATION` keyword argument:

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
            data=payload,
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, 200)
        
        data = response.json()
        self.assertIn('token', data)
        self.assertTrue(len(data['token']) > 0)
```

The `HTTP_AUTHORIZATION` keyword maps to the `Authorization` HTTP header. The `Bearer ` prefix followed by the token key matches the format the authentication middleware expects.

## Test Helper Methods

A base test class with helper methods reduces boilerplate across your test suite:

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
            data={'title': 'Test', 'body': 'Content'},
            content_type='application/json'
        )
        
        self.assertAPISuccess(response)
```

The `api_client()` helper creates an authenticated client in one call, and `assertAPISuccess` / `assertAPIError` make assertions more readable and consistent.

## Common Pitfalls

1. **Only testing the happy path**: Testing only successful requests misses validation errors, auth failures, and edge cases.

2. **Testing implementation, not behavior**: Tests should verify response format and status codes, not internal implementation details.

3. **Slow test setup**: Creating too much data in `setUp()` slows down test suites. Use `setUpTestData()` for shared data.

## Best Practices

1. **Test all HTTP methods**: Verify GET, POST, PUT, PATCH, and DELETE for each endpoint.

2. **Test error responses**: Verify 400, 401, 403, 404, and 405 responses with correct error formats.

3. **Use setUpTestData for shared data**: It runs once per class, not once per test.

4. **Name tests descriptively**: `test_create_article_without_title_returns_400` is better than `test_create_error`.

## Summary

Django's test client enables comprehensive API testing without a running server. Test all HTTP methods, verify both success and error responses, and use `setUpTestData()` for efficient shared data. Descriptive test names serve as documentation for expected API behavior.

## Code Examples

**Basic API test case with Django's test client**

```python
from django.test import TestCase, Client
from django.contrib.auth.models import User

class ArticleAPITests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user('testuser', password='testpass')
        self.article = Article.objects.create(title='Test', author=self.user)

    def test_list_articles(self):
        response = self.client.get('/api/articles/')
        self.assertEqual(response.status_code, 200)
        data = response.json()
        self.assertEqual(len(data['articles']), 1)

    def test_create_requires_auth(self):
        response = self.client.post('/api/articles/',
            data={'title': 'New'}, content_type='application/json')
        self.assertEqual(response.status_code, 401)
```


## Resources

- [Testing in Django](https://docs.djangoproject.com/en/6.0/topics/testing/) — Official Django testing documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*