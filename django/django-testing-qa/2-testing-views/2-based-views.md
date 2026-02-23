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

## Real World Context

Most modern Django applications use class-based views extensively. Testing them through URLs is preferred because it exercises the full middleware stack, URL routing, and template rendering. RequestFactory is reserved for cases where you need to test a single view method in isolation, such as a custom `get_queryset()` override.

## Common Pitfalls

1. **Testing CBV internals instead of behavior**: Do not test that `get_context_data` returns a specific dict. Test that the rendered page contains expected content.
2. **Forgetting to set request.user with RequestFactory**: RequestFactory does not process middleware, so you must manually set the user attribute.
3. **Not testing permission mixins**: LoginRequiredMixin and PermissionRequiredMixin need explicit tests for both allowed and denied access.

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