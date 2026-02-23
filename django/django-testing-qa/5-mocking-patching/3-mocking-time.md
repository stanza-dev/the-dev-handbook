---
source_course: "django-testing-qa"
source_lesson: "django-testing-qa-mocking-time"
---

# Mocking Time and Dates

## Introduction

Time-dependent code needs controlled dates for reliable testing. Freezegun makes this easy.

## Key Concepts

**Freeze Time**: Lock the clock to a specific moment.

**Time Travel**: Move through time in tests.

## Deep Dive

### Using Freezegun

```python
from freezegun import freeze_time
from django.utils import timezone

class TimeBasedTests(TestCase):
    @freeze_time('2024-06-15 12:00:00')
    def test_article_is_recent(self):
        article = Article.objects.create(
            title='Test',
            pub_date=timezone.datetime(2024, 6, 10, tzinfo=timezone.utc)
        )
        self.assertTrue(article.is_recent)  # Within 7 days
    
    @freeze_time('2024-06-15 12:00:00')
    def test_article_not_recent(self):
        article = Article.objects.create(
            title='Test',
            pub_date=timezone.datetime(2024, 1, 1, tzinfo=timezone.utc)
        )
        self.assertFalse(article.is_recent)
```

### Testing Expiration

```python
class TokenExpirationTests(TestCase):
    @freeze_time('2024-01-01 00:00:00')
    def test_token_valid_before_expiry(self):
        token = Token.objects.create(
            user=self.user,
            expires_at=timezone.datetime(2024, 1, 2, tzinfo=timezone.utc)
        )
        self.assertTrue(token.is_valid)
    
    @freeze_time('2024-01-03 00:00:00')
    def test_token_invalid_after_expiry(self):
        token = Token.objects.create(
            user=self.user,
            expires_at=timezone.datetime(2024, 1, 2, tzinfo=timezone.utc)
        )
        self.assertFalse(token.is_valid)
```

### Time Travel

```python
from freezegun import freeze_time

class ScheduledTaskTests(TestCase):
    def test_task_runs_at_scheduled_time(self):
        with freeze_time('2024-01-01 08:00:00') as frozen:
            task = ScheduledTask.objects.create(
                run_at=timezone.datetime(2024, 1, 1, 9, tzinfo=timezone.utc)
            )
            self.assertFalse(task.should_run)
            
            frozen.move_to('2024-01-01 09:00:00')
            self.assertTrue(task.should_run)
```

## Real World Context

Time-dependent features are everywhere: subscription expiration, trial periods, scheduled tasks, rate limiting, and 'posted 3 hours ago' displays. Testing these without freezegun requires fragile approaches like sleeping or using approximate assertions. Freezegun makes time deterministic, so tests always produce the same result.

## Common Pitfalls

1. **Using naive datetimes**: Always use timezone-aware datetimes with `tzinfo=timezone.utc`. Naive datetimes cause subtle bugs in time comparisons.
2. **Not testing boundary conditions**: If a token expires after 24 hours, test at 23:59 and 24:01, not just 'before' and 'after'.
3. **Forgetting that freezegun affects all code**: When time is frozen, auto_now and auto_now_add fields also use the frozen time.

## Best Practices

1. **Use timezone-aware datetimes**: Always include tzinfo.
2. **Test boundaries**: Test just before/after transitions.
3. **Install freezegun**: `pip install freezegun`.

## Summary

Use freezegun to control time in tests. Test time-sensitive features like expiration, scheduling, and recency checks. Always use timezone-aware datetimes.

## Resources

- [Freezegun](https://github.com/spulec/freezegun) — Freezegun documentation

---

> 📘 *This lesson is part of the [Django Testing & Quality Assurance](https://stanza.dev/courses/django-testing-qa) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*