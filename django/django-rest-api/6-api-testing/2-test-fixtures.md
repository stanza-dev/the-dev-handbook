---
source_course: "django-rest-api"
source_lesson: "django-rest-api-test-fixtures"
---

# Test Fixtures and Factories

## Introduction

Good tests need consistent, realistic data. Fixtures and factories help you create test data efficiently without repetitive setup code.

## Key Concepts

**Fixture**: Pre-defined data loaded before tests run. Can be JSON, YAML, or created programmatically.

**Factory**: A pattern for creating test objects with sensible defaults and easy customization.

**setUp/setUpTestData**: Methods to create data before each test or once per test class.

## Deep Dive

### Using setUpTestData (Efficient)

```python
from django.test import TestCase

class ArticleAPITests(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Created once for all tests in this class (faster)
        cls.user = User.objects.create_user('testuser', 'test@test.com', 'pass')
        cls.article = Article.objects.create(
            title='Test Article',
            body='Content',
            author=cls.user
        )
    
    def test_get_article(self):
        response = self.client.get(f'/api/articles/{self.article.id}/')
        self.assertEqual(response.status_code, 200)
```

### Factory Pattern

```python
# factories.py
import factory
from .models import User, Article

class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f'user{n}')
    email = factory.LazyAttribute(lambda obj: f'{obj.username}@test.com')
    password = factory.PostGenerationMethodCall('set_password', 'testpass')

class ArticleFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Article
    
    title = factory.Sequence(lambda n: f'Article {n}')
    body = factory.Faker('paragraph')
    author = factory.SubFactory(UserFactory)

# Usage in tests
class ArticleTests(TestCase):
    def test_list_articles(self):
        ArticleFactory.create_batch(5)
        response = self.client.get('/api/articles/')
        self.assertEqual(len(response.json()['data']), 5)
```

### Custom Test Mixins

```python
class APITestMixin:
    def create_authenticated_client(self, user=None):
        if user is None:
            user = UserFactory()
        token = APIToken.objects.create(user=user)
        self.client.defaults['HTTP_AUTHORIZATION'] = f'Bearer {token.key}'
        return user
    
    def assertJsonResponse(self, response, expected_status=200):
        self.assertEqual(response.status_code, expected_status)
        self.assertEqual(response['Content-Type'], 'application/json')
        return response.json()
```

## Best Practices

1. **Use setUpTestData for shared data**: It's faster than setUp.
2. **Use factories for complex objects**: They handle relationships cleanly.
3. **Keep test data minimal**: Create only what the test needs.

## Summary

Use setUpTestData for efficient shared data, factories for flexible object creation, and custom mixins to reduce boilerplate in your API tests.

## Resources

- [Django Test Tools](https://docs.djangoproject.com/en/6.0/topics/testing/tools/) — Django testing tools reference

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*