---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-integration-testing"
---

# Integration Test Strategies

## Introduction

Integration tests verify that multiple components of your application work together correctly. While unit tests check individual pieces in isolation, integration tests exercise the full request cycle: URL routing, middleware, views, forms, models, and templates working as a system.

## Key Concepts

**Integration Test**: A test that exercises multiple components working together through a realistic scenario.

**LiveServerTestCase**: Starts a real HTTP server for browser-based testing with Selenium.

**mail.outbox**: Django's test email backend that captures sent emails in a list for verification.

## Real World Context

Integration tests catch bugs that unit tests miss -- like a view that calls a service with the wrong arguments, or a template that references a missing context variable. In practice, a healthy Django test suite has about 70% unit tests and 30% integration tests. Integration tests are slower but provide confidence that your features actually work end-to-end.

## Deep Dive

## Full Request Cycle Test

```python
from django.test import TestCase, TransactionTestCase
from django.urls import reverse


class ArticleWorkflowTests(TestCase):
    """Test complete article workflow."""
    
    @classmethod
    def setUpTestData(cls):
        cls.author = User.objects.create_user(
            'author', 'author@test.com', 'pass',
            is_staff=True
        )
        cls.editor = User.objects.create_user(
            'editor', 'editor@test.com', 'pass',
            is_staff=True
        )
        cls.reader = User.objects.create_user(
            'reader', 'reader@test.com', 'pass'
        )
        
        # Set up permissions
        from django.contrib.auth.models import Permission
        publish_perm = Permission.objects.get(codename='publish_article')
        cls.editor.user_permissions.add(publish_perm)
    
    def test_complete_article_lifecycle(self):
        """Test article from creation to publication."""
        # Step 1: Author creates article
        self.client.force_login(self.author)
        response = self.client.post(reverse('article-create'), {
            'title': 'Integration Test Article',
            'body': 'This is the content.',
            'status': 'draft',
        })
        self.assertEqual(response.status_code, 302)
        
        article = Article.objects.get(title='Integration Test Article')
        self.assertEqual(article.status, 'draft')
        self.assertEqual(article.author, self.author)
        
        # Step 2: Article not visible to readers
        self.client.force_login(self.reader)
        response = self.client.get(reverse('article-list'))
        self.assertNotContains(response, 'Integration Test Article')
        
        # Step 3: Editor publishes article
        self.client.force_login(self.editor)
        response = self.client.post(
            reverse('article-publish', kwargs={'pk': article.pk})
        )
        self.assertEqual(response.status_code, 302)
        
        article.refresh_from_db()
        self.assertEqual(article.status, 'published')
        
        # Step 4: Reader can now see article
        self.client.force_login(self.reader)
        response = self.client.get(reverse('article-list'))
        self.assertContains(response, 'Integration Test Article')
        
        # Step 5: Reader can view detail
        response = self.client.get(
            reverse('article-detail', kwargs={'slug': article.slug})
        )
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'This is the content.')
```

## Testing with Multiple Models

```python
class CommentIntegrationTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user('test', 'test@test.com', 'pass')
        self.article = Article.objects.create(
            title='Test Article',
            body='Content',
            status='published',
            author=self.user
        )
    
    def test_comment_flow(self):
        """Test commenting on an article."""
        self.client.force_login(self.user)
        
        # Post comment
        response = self.client.post(
            reverse('comment-create', kwargs={'article_pk': self.article.pk}),
            {'body': 'Great article!'}
        )
        self.assertEqual(response.status_code, 302)
        
        # Verify comment exists
        self.assertEqual(self.article.comments.count(), 1)
        comment = self.article.comments.first()
        self.assertEqual(comment.body, 'Great article!')
        self.assertEqual(comment.author, self.user)
        
        # Comment appears on article page
        response = self.client.get(
            reverse('article-detail', kwargs={'slug': self.article.slug})
        )
        self.assertContains(response, 'Great article!')
```

## Testing Email Integration

```python
from django.core import mail


class EmailIntegrationTests(TestCase):
    def test_registration_sends_welcome_email(self):
        """Test that registration sends a welcome email."""
        response = self.client.post(reverse('register'), {
            'username': 'newuser',
            'email': 'new@example.com',
            'password1': 'complexpass123',
            'password2': 'complexpass123',
        })
        
        # Check redirect (successful registration)
        self.assertEqual(response.status_code, 302)
        
        # Verify email sent
        self.assertEqual(len(mail.outbox), 1)
        email = mail.outbox[0]
        self.assertEqual(email.subject, 'Welcome to Our Site!')
        self.assertEqual(email.to, ['new@example.com'])
        self.assertIn('newuser', email.body)
    
    def test_password_reset_flow(self):
        """Test complete password reset flow."""
        user = User.objects.create_user('test', 'test@test.com', 'oldpass')
        
        # Request reset
        response = self.client.post(reverse('password_reset'), {
            'email': 'test@test.com'
        })
        self.assertEqual(response.status_code, 302)
        
        # Get token from email
        self.assertEqual(len(mail.outbox), 1)
        email_body = mail.outbox[0].body
        
        # Extract reset link (simplified)
        import re
        match = re.search(r'/reset/(\S+)/(\S+)/', email_body)
        uidb64, token = match.groups()
        
        # Complete reset
        response = self.client.post(
            reverse('password_reset_confirm', kwargs={
                'uidb64': uidb64,
                'token': token
            }),
            {'new_password1': 'newpass123', 'new_password2': 'newpass123'}
        )
        
        # Verify new password works
        user.refresh_from_db()
        self.assertTrue(user.check_password('newpass123'))
```

## LiveServerTestCase with Selenium

```python
from django.test import LiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


class BrowserTests(LiveServerTestCase):
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.browser = webdriver.Chrome()
        cls.browser.implicitly_wait(10)
    
    @classmethod
    def tearDownClass(cls):
        cls.browser.quit()
        super().tearDownClass()
    
    def test_user_can_login(self):
        """Test login flow in real browser."""
        User.objects.create_user('testuser', 'test@test.com', 'testpass')
        
        # Navigate to login page
        self.browser.get(f'{self.live_server_url}/login/')
        
        # Fill in form
        username_input = self.browser.find_element(By.NAME, 'username')
        username_input.send_keys('testuser')
        
        password_input = self.browser.find_element(By.NAME, 'password')
        password_input.send_keys('testpass')
        
        # Submit
        submit_btn = self.browser.find_element(By.CSS_SELECTOR, 'button[type=submit]')
        submit_btn.click()
        
        # Wait for redirect and verify
        WebDriverWait(self.browser, 10).until(
            EC.url_contains('/dashboard/')
        )
        
        self.assertIn('Welcome', self.browser.page_source)
```

## Common Pitfalls

1. **Making integration tests too broad**: Each test should verify one workflow or scenario, not the entire application.
2. **Forgetting to call refresh_from_db()**: After actions that modify the database, the in-memory object is stale.
3. **Not cleaning up Selenium browsers**: Always quit the browser in tearDownClass to avoid resource leaks.

## Best Practices

1. **Test complete user workflows**: Create, read, update, delete cycles test the full stack.
2. **Use django.core.mail.outbox for email tests**: Django captures emails in tests automatically.
3. **Run Selenium tests headless in CI**: Use `--headless` Chrome option for faster, display-free testing.

## Summary

- Integration tests verify components work together through realistic scenarios
- Test complete user workflows from authentication through data modification
- Use django.core.mail.outbox to verify email sending
- LiveServerTestCase enables Selenium browser testing
- Balance integration tests with unit tests for speed and coverage

## Code Examples

**An integration test that creates an article via POST and verifies it appears in the list view.**

```python
from django.test import TestCase
from django.urls import reverse
from django.contrib.auth.models import User


class ArticleWorkflowTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            'author', 'author@test.com', 'pass'
        )

    def test_create_and_view_article(self):
        self.client.force_login(self.user)
        # Create article
        self.client.post(reverse('article-create'), {
            'title': 'Integration Test',
            'body': 'Content',
            'status': 'published',
        })
        # Verify it appears in list
        response = self.client.get(reverse('article-list'))
        self.assertContains(response, 'Integration Test')

```


## Resources

- [Integration Testing](https://docs.djangoproject.com/en/6.0/topics/testing/tools/#liveservertestcase) — Live server testing

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*