---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-selenium-browser-tests"
---

# Browser Testing with Selenium

## Introduction

Selenium tests verify your application works correctly in a real browser, testing JavaScript and user interactions.

## Key Concepts

**WebDriver**: Controls the browser programmatically.

**LiveServerTestCase**: Runs a real HTTP server for Selenium.

## Deep Dive

### Basic Selenium Test

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
    
    def test_login_flow(self):
        User.objects.create_user('test', 'test@test.com', 'pass')
        
        self.browser.get(f'{self.live_server_url}/login/')
        
        self.browser.find_element(By.NAME, 'username').send_keys('test')
        self.browser.find_element(By.NAME, 'password').send_keys('pass')
        self.browser.find_element(By.CSS_SELECTOR, 'button[type=submit]').click()
        
        WebDriverWait(self.browser, 10).until(
            EC.url_contains('/dashboard/')
        )
        
        self.assertIn('Dashboard', self.browser.title)
```

### Testing JavaScript Interactions

```python
class JavaScriptTests(LiveServerTestCase):
    def test_dynamic_form(self):
        self.browser.get(f'{self.live_server_url}/form/')
        
        # Click button that shows hidden field
        self.browser.find_element(By.ID, 'show-more').click()
        
        # Wait for element to be visible
        element = WebDriverWait(self.browser, 5).until(
            EC.visibility_of_element_located((By.ID, 'extra-field'))
        )
        
        self.assertTrue(element.is_displayed())
```

### Headless Mode

```python
from selenium.webdriver.chrome.options import Options

@classmethod
def setUpClass(cls):
    super().setUpClass()
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--no-sandbox')
    cls.browser = webdriver.Chrome(options=options)
```

## Real World Context

Selenium tests are the most realistic form of testing -- they use a real browser just like a user would. They are essential for testing JavaScript interactions, dynamic forms, and complex UI workflows. However, they are also the slowest and most fragile tests, so they should be reserved for critical user paths like login, checkout, and key business workflows.

## Common Pitfalls

1. **Using time.sleep() instead of explicit waits**: sleep() either waits too long (slow) or not long enough (flaky). Always use WebDriverWait.
2. **Not running headless in CI**: Browser tests fail in CI environments that lack a display unless you use headless mode.
3. **Writing too many Selenium tests**: Browser tests are 10-100x slower than unit tests. Reserve them for critical paths that cannot be tested otherwise.

## Best Practices

1. **Use explicit waits**: More reliable than implicit waits.
2. **Run headless in CI**: Faster and doesn't need display.
3. **Keep tests focused**: Browser tests are slow, test critical paths only.

## Summary

Use Selenium with LiveServerTestCase for browser testing. Use explicit waits for reliability. Run headless in CI. Keep browser tests focused on critical user journeys.

## Resources

- [Selenium Python](https://selenium-python.readthedocs.io/) — Selenium Python documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*