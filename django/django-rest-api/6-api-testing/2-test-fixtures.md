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

## Real World Context

Test fixtures and factories are essential for:
- **Large test suites**: Dozens of test classes sharing common setup patterns
- **Complex data models**: Models with many relationships that need consistent test data
- **Team collaboration**: Factories provide a shared vocabulary for creating test objects
- **CI/CD pipelines**: Efficient test data creation keeps test suites fast

## Deep Dive

### Using setUpTestData (Efficient)

`setUpTestData` runs once per test class (not per test method), making it significantly faster for shared read-only data:

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

Because `setUpTestData` uses a class method, the data persists across all tests in the class. Django wraps each test in a transaction savepoint, so modifications are rolled back between tests.

### Factory Pattern

The `factory_boy` library generates test objects with sensible defaults. `Sequence` ensures unique values, `SubFactory` handles relationships, and `create_batch` generates multiple instances:

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

`ArticleFactory.create_batch(5)` creates five articles in a single call, each with an auto-generated author via `SubFactory(UserFactory)`. This is far more concise than manual `create()` calls.

### Custom Test Mixins

Mixins add reusable helper methods to your test classes without duplicating code across test files:

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

`create_authenticated_client()` handles token creation and header setup in one call. `assertJsonResponse()` combines status and content-type checks into a single assertion.

## Common Pitfalls

1. **Brittle fixtures**: JSON fixtures break when models change. Prefer programmatic data creation.

2. **Shared mutable state**: Using `setUp()` when `setUpTestData()` would suffice wastes time recreating identical data.

3. **Over-specifying test data**: Creating more data than a test needs makes tests harder to understand and slower to run.

## Best Practices

1. **Use setUpTestData for shared data**: It's faster than setUp.
2. **Use factories for complex objects**: They handle relationships cleanly.
3. **Keep test data minimal**: Create only what the test needs.

## Summary

Use setUpTestData for efficient shared data, factories for flexible object creation, and custom mixins to reduce boilerplate in your API tests.

## Code Examples

**Efficient test data setup with setUpTestData for class-level fixtures**

```python
from django.test import TestCase

class ArticleAPITests(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Created once for all tests (efficient)
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


## Resources

- [Django Test Tools](https://docs.djangoproject.com/en/6.0/topics/testing/tools/) — Django testing tools reference

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*