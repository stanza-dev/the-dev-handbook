---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-testing-class-based-views"
---

# Testing Class-Based Views

## Introduction

Class-based views (CBVs) require special testing techniques. You can test them via the URL or directly instantiate them.

## Key Concepts

**View Instance**: Direct access to view logic.

**Request Factory**: Creates request objects for direct view testing.

## Deep Dive

### Testing via URL (Recommended)

```python
class ArticleListViewTests(TestCase):
    def test_list_view_status_code(self):
        response = self.client.get(reverse('article-list'))
        self.assertEqual(response.status_code, 200)
    
    def test_list_view_template(self):
        response = self.client.get(reverse('article-list'))
        self.assertTemplateUsed(response, 'articles/list.html')
    
    def test_list_view_context(self):
        Article.objects.create(title='Test')
        response = self.client.get(reverse('article-list'))
        self.assertEqual(len(response.context['articles']), 1)
```

### Using RequestFactory

```python
from django.test import RequestFactory

class ArticleDetailViewTests(TestCase):
    def setUp(self):
        self.factory = RequestFactory()
        self.user = User.objects.create_user('test', 'test@test.com', 'pass')
    
    def test_view_directly(self):
        article = Article.objects.create(title='Test')
        request = self.factory.get('/articles/test/')
        request.user = self.user
        
        response = ArticleDetailView.as_view()(request, slug='test')
        self.assertEqual(response.status_code, 200)
```

### Testing Mixins

```python
class LoginRequiredViewTests(TestCase):
    def test_redirects_anonymous_user(self):
        response = self.client.get(reverse('dashboard'))
        self.assertRedirects(response, '/login/?next=/dashboard/')
    
    def test_allows_authenticated_user(self):
        self.client.force_login(self.user)
        response = self.client.get(reverse('dashboard'))
        self.assertEqual(response.status_code, 200)
```

## Best Practices

1. **Test via URL first**: Most realistic way to test views.
2. **Use RequestFactory for unit tests**: When testing view logic in isolation.
3. **Test all HTTP methods**: GET, POST, PUT, DELETE as appropriate.

## Summary

Test CBVs primarily through URLs for integration testing. Use RequestFactory for unit testing specific view methods. Always test authentication and permission requirements.

## Resources

- [Testing CBVs](https://docs.djangoproject.com/en/6.0/topics/testing/advanced/#the-request-factory) — RequestFactory documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*