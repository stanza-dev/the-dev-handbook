---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-testing-redirects-errors"
---

# Testing Redirects and Error Responses

## Introduction

Properly testing redirects and error handling ensures your application behaves correctly in all scenarios.

## Key Concepts

**Redirect**: HTTP 3xx response pointing to another URL.

**Error Response**: HTTP 4xx/5xx status codes.

## Deep Dive

### Testing Redirects

```python
class RedirectTests(TestCase):
    def test_successful_form_redirects(self):
        self.client.force_login(self.user)
        response = self.client.post(reverse('article-create'), {
            'title': 'New Article',
            'body': 'Content',
        })
        article = Article.objects.get(title='New Article')
        self.assertRedirects(
            response,
            reverse('article-detail', args=[article.slug])
        )
    
    def test_redirect_chain(self):
        response = self.client.get('/old-url/', follow=True)
        self.assertEqual(response.redirect_chain, [
            ('/new-url/', 301),
        ])
```

### Testing 404 Errors

```python
class ErrorTests(TestCase):
    def test_invalid_article_returns_404(self):
        response = self.client.get('/articles/nonexistent/')
        self.assertEqual(response.status_code, 404)
    
    def test_404_uses_custom_template(self):
        response = self.client.get('/articles/nonexistent/')
        self.assertTemplateUsed(response, '404.html')
```

### Testing Permission Denied

```python
class PermissionTests(TestCase):
    def test_unauthorized_returns_403(self):
        other_user = User.objects.create_user('other', 'o@t.com', 'pass')
        self.client.force_login(other_user)
        
        response = self.client.get(reverse('admin-only'))
        self.assertEqual(response.status_code, 403)
```

### Testing with follow=True

```python
class FollowRedirectTests(TestCase):
    def test_login_then_redirect(self):
        response = self.client.post(
            reverse('login'),
            {'username': 'test', 'password': 'pass'},
            follow=True  # Follow all redirects
        )
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'dashboard.html')
```

## Real World Context

Redirect and error handling tests are often overlooked but are critical for user experience. A broken redirect after form submission leaves users staring at a blank page. Missing 404 handlers expose Django's debug page in production. These tests catch issues that are hard to spot in development but immediately visible to users.

## Common Pitfalls

1. **Not testing the full redirect chain**: Use `follow=True` to verify the final destination, not just the first redirect.
2. **Forgetting to test custom error templates**: Django serves default error pages unless you provide custom 404.html and 500.html templates.
3. **Checking only status codes**: A 200 response might still show an error message in the body. Always check content too.

## Best Practices

1. **Test both status code and destination**: Use assertRedirects.
2. **Use follow=True sparingly**: Be explicit about redirect chains.
3. **Test custom error pages**: Ensure 404/500 pages work.

## Summary

Use assertRedirects for redirect testing. Test 404, 403, and other error responses explicitly. Use follow=True when you need to test the final destination.

## Resources

- [Testing Redirects](https://docs.djangoproject.com/en/6.0/topics/testing/tools/#django.test.SimpleTestCase.assertRedirects) — assertRedirects documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*