---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-on-commit-hooks"
---

# on_commit Hooks

Sometimes you need to perform actions only after a transaction successfully commits—like sending emails or queuing background tasks.

## The Problem

```python
from django.db import transaction

def create_order(cart, user):
    with transaction.atomic():
        order = Order.objects.create(user=user)
        # DON'T DO THIS:
        send_order_confirmation_email(order)  # What if transaction rolls back?
        return order
```

If the transaction rolls back after sending the email, the user gets a confirmation for an order that doesn't exist!

## The Solution: on_commit

```python
from django.db import transaction

def create_order(cart, user):
    with transaction.atomic():
        order = Order.objects.create(user=user)
        # ... more order logic ...
        
        # Schedule email for after commit
        transaction.on_commit(
            lambda: send_order_confirmation_email(order.id)
        )
        
        return order
```

## How on_commit Works

1. Registers a callback function
2. Callback only runs if the transaction commits
3. If transaction rolls back, callback is discarded
4. Callbacks run in the order registered

## Common Use Cases

### Sending Notifications

To fire a signal, call `.send()` with the sender class and any keyword arguments your receivers expect:

```python
from django.db import transaction
from .tasks import send_notification

@transaction.atomic
def accept_friend_request(request_id):
    friend_request = FriendRequest.objects.get(pk=request_id)
    friend_request.status = 'accepted'
    friend_request.save()
    
    # Create friendship
    Friendship.objects.create(
        user1=friend_request.from_user,
        user2=friend_request.to_user
    )
    
    # Only notify after everything succeeds
    transaction.on_commit(
        lambda: send_notification(
            friend_request.from_user.id,
            f"{friend_request.to_user.username} accepted your request!"
        )
    )
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Queuing Background Tasks

The following example demonstrates how to use queuing background tasks in practice:

```python
from django.db import transaction
from .tasks import process_upload, generate_thumbnails

@transaction.atomic
def handle_upload(file, user):
    upload = Upload.objects.create(
        file=file,
        user=user,
        status='pending'
    )
    
    # Queue tasks only if upload record is committed
    transaction.on_commit(
        lambda: process_upload.delay(upload.id)
    )
    transaction.on_commit(
        lambda: generate_thumbnails.delay(upload.id)
    )
    
    return upload
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Cache Invalidation

The following example demonstrates how to use cache invalidation in practice:

```python
from django.db import transaction
from django.core.cache import cache

@transaction.atomic
def update_product(product_id, data):
    product = Product.objects.select_for_update().get(pk=product_id)
    for key, value in data.items():
        setattr(product, key, value)
    product.save()
    
    # Invalidate cache after commit
    transaction.on_commit(
        lambda: cache.delete(f'product:{product_id}')
    )
    transaction.on_commit(
        lambda: cache.delete(f'category:{product.category_id}:products')
    )
```

## Nested Transactions

With nested `atomic()` blocks, `on_commit` only runs when the **outermost** transaction commits. Callbacks run in **registration order**:

```python
from django.db import transaction

with transaction.atomic():  # Outer
    User.objects.create(username='outer')
    transaction.on_commit(lambda: print('Outer committed'))  # Registered first
    
    with transaction.atomic():  # Inner (savepoint)
        User.objects.create(username='inner')
        transaction.on_commit(lambda: print('Inner committed'))  # Registered second

# Output (registration order):
# Outer committed
# Inner committed
```

## Testing on_commit

In tests, use `TestCase.captureOnCommitCallbacks`:

```python
from django.test import TestCase

class OrderTests(TestCase):
    def test_order_sends_email(self):
        with self.captureOnCommitCallbacks(execute=True) as callbacks:
            order = create_order(self.cart, self.user)
        
        # Callbacks were executed
        self.assertEqual(len(mail.outbox), 1)
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test on_commit hooks with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some on_commit hooks features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- on_commit Hooks is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from on_commit Hooks**

```python
from django.db import transaction

def create_order(cart, user):
    with transaction.atomic():
        order = Order.objects.create(user=user)
        # DON'T DO THIS:
        send_order_confirmation_email(order)  # What if transaction rolls back?
        return order
```


## Resources

- [transaction.on_commit](https://docs.djangoproject.com/en/6.0/topics/db/transactions/#performing-actions-after-commit) — Official documentation on on_commit hooks

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*