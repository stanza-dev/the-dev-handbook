---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-savepoints"
---

# Savepoints and Nested Transactions

## Introduction

Nested `transaction.atomic()` blocks create savepoints — intermediate checkpoints within a transaction. If the inner block fails, only its changes are rolled back while the outer transaction continues.

## Key Concepts

- **Savepoint**: A named checkpoint within a transaction that you can roll back to without aborting the entire transaction.
- **Nested atomic**: An `atomic()` block inside another `atomic()` block creates a savepoint rather than a new transaction.
- **`on_commit`**: Callbacks registered inside a nested atomic only fire when the outermost transaction commits.

## Real World Context

Processing a batch of orders where each order should succeed or fail independently: wrap the batch in an outer `atomic()`, each order in an inner `atomic()`. If one order fails, the others still commit.

## Deep Dive

### Nested Atomic Blocks

When you nest `atomic()` blocks, the inner block creates a savepoint rather than a new transaction. If it fails, only its changes are rolled back:

```python
from django.db import transaction

with transaction.atomic():  # Outer: real transaction
    Account.objects.filter(pk=1).update(balance=1000)

    try:
        with transaction.atomic():  # Inner: savepoint
            Account.objects.filter(pk=2).update(balance=-500)
            raise ValueError('Something went wrong')
    except ValueError:
        pass  # Savepoint rolled back, pk=2 unchanged

    # pk=1 update is still intact
    Account.objects.filter(pk=3).update(balance=500)
# Both pk=1 and pk=3 updates commit
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Manual Savepoint Control

For finer control, you can manually create, commit, and roll back savepoints using Django's transaction API:

```python
from django.db import transaction

with transaction.atomic():
    item.save()

    sid = transaction.savepoint()
    try:
        risky_operation()
    except Exception:
        transaction.savepoint_rollback(sid)
    else:
        transaction.savepoint_commit(sid)

    # Continue with the transaction
    log.save()
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Batch Processing Pattern

This pattern processes a list of items where each can succeed or fail independently. The outer transaction wraps the batch, while inner savepoints isolate each item:

```python
def process_orders(order_ids):
    results = {'success': [], 'failed': []}

    with transaction.atomic():  # Outer transaction
        for order_id in order_ids:
            try:
                with transaction.atomic():  # Savepoint per order
                    order = Order.objects.select_for_update().get(pk=order_id)
                    process_single_order(order)
                    results['success'].append(order_id)
            except Exception as e:
                results['failed'].append((order_id, str(e)))
                # Savepoint rolled back, continue with next

    return results
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### on_commit with Savepoints

Callbacks registered with `on_commit` inside a savepoint only fire when the outermost transaction commits — not when the savepoint itself commits:

```python
with transaction.atomic():  # Outer
    user.save()

    with transaction.atomic():  # Savepoint
        profile.save()
        transaction.on_commit(lambda: send_welcome_email(user.pk))

    # on_commit callback fires only when outer transaction commits
```

## Common Pitfalls

1. **Assuming nested atomic = nested transaction** — Most databases don't support true nested transactions. Django uses savepoints to emulate them.
2. **Catching exceptions outside the atomic block** — If an exception escapes the `with atomic()` block, the entire transaction is rolled back. Always catch inside.
3. **Relying on on_commit inside savepoints** — `on_commit` callbacks from a rolled-back savepoint are discarded. They only fire if the savepoint commits AND the outer transaction commits.

## Best Practices

1. **Use nested atomic for independent operations** — If one step can fail without invalidating the others, wrap it in a nested atomic.
2. **Prefer automatic savepoints over manual** — `with transaction.atomic()` is cleaner and less error-prone than manual `savepoint()`/`savepoint_commit()`.
3. **Test with `TransactionTestCase`** — Django's `TestCase` wraps tests in transactions, hiding savepoint behavior. Use `TransactionTestCase` for transaction-related tests.

## Summary

- Nested `atomic()` blocks create savepoints, not new transactions.
- If a savepoint fails, only its changes roll back — the outer transaction continues.
- `on_commit` callbacks from savepoints only fire when the outermost transaction commits.
- Manual savepoints (`savepoint()`, `savepoint_rollback()`) give fine-grained control.
- Use nested atomic for batch processing where individual items can fail independently.

## Code Examples

**Nested atomic blocks create savepoints — inner failures don't roll back the outer transaction**

```python
from django.db import transaction

with transaction.atomic():  # Real transaction
    Account.objects.filter(pk=1).update(balance=1000)

    try:
        with transaction.atomic():  # Savepoint
            Account.objects.filter(pk=2).update(balance=-500)
            raise ValueError('Bad data')
    except ValueError:
        pass  # Only pk=2 rolled back

    Account.objects.filter(pk=3).update(balance=500)
# pk=1 and pk=3 committed; pk=2 rolled back
```


## Resources

- [Savepoints](https://docs.djangoproject.com/en/6.0/topics/db/transactions/#savepoints) — Official Django documentation on savepoints

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*