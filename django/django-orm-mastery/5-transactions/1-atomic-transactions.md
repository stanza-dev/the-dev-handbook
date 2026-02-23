---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-atomic-transactions"
---

# Atomic Transactions

Transactions ensure that a group of database operations either all succeed or all fail together. Django provides the `atomic()` context manager and decorator.

## Why Transactions?

Consider transferring money between accounts:

```python
# WITHOUT transactions - DANGEROUS!
def transfer_money(from_account, to_account, amount):
    from_account.balance -= amount
    from_account.save()
    # What if server crashes here?
    to_account.balance += amount
    to_account.save()
```

If the server crashes after the first save, money disappears!

## Using atomic()

```python
from django.db import transaction

# As a context manager
def transfer_money(from_account, to_account, amount):
    with transaction.atomic():
        from_account.balance -= amount
        from_account.save()
        # If anything fails, both saves are rolled back
        to_account.balance += amount
        to_account.save()

# As a decorator
@transaction.atomic
def transfer_money(from_account, to_account, amount):
    from_account.balance -= amount
    from_account.save()
    to_account.balance += amount
    to_account.save()
```

## How atomic() Works

1. Starts a transaction (or savepoint if already in one)
2. Executes your code
3. **Success**: Commits all changes
4. **Exception**: Rolls back all changes

```python
from django.db import transaction

def create_order(cart, user):
    with transaction.atomic():
        # Create order
        order = Order.objects.create(user=user)
        
        # Create order items
        for item in cart.items.all():
            OrderItem.objects.create(
                order=order,
                product=item.product,
                quantity=item.quantity
            )
            # Decrease stock
            item.product.stock -= item.quantity
            item.product.save()
        
        # Clear cart
        cart.items.all().delete()
        
        # If ANY of these fail, EVERYTHING rolls back
        return order
```

## Nested atomic Blocks

Nested atomic blocks create savepoints:

```python
from django.db import transaction

def process_order(order):
    with transaction.atomic():  # Outer transaction
        order.status = 'processing'
        order.save()
        
        try:
            with transaction.atomic():  # Savepoint
                # Try to charge the card
                charge_credit_card(order)
                order.status = 'paid'
                order.save()
        except PaymentError:
            # Savepoint rolled back, but outer transaction continues
            order.status = 'payment_failed'
            order.save()
        
        # Outer transaction commits with final status
```

## Handling Exceptions

```python
from django.db import transaction, IntegrityError

def create_user_profile(user_data):
    try:
        with transaction.atomic():
            user = User.objects.create(**user_data)
            Profile.objects.create(user=user)
            return user
    except IntegrityError:
        # Handle duplicate user
        return None
```

## select_for_update() - Row Locking

Prevent race conditions by locking rows:

```python
from django.db import transaction

def deduct_stock(product_id, quantity):
    with transaction.atomic():
        # Lock the row until transaction completes
        product = Product.objects.select_for_update().get(pk=product_id)
        
        if product.stock >= quantity:
            product.stock -= quantity
            product.save()
            return True
        return False
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### select_for_update Options

The following example demonstrates how to use select_for_update options in practice:

```python
# Basic lock
Product.objects.select_for_update().get(pk=1)

# Skip locked rows (don't wait)
Product.objects.select_for_update(skip_locked=True).filter(stock__gt=0)

# No wait (raise error if locked)
Product.objects.select_for_update(nowait=True).get(pk=1)

# Lock specific tables in multi-table queries
Order.objects.select_for_update(of=('self', 'customer')).get(pk=1)
```

## Django's Default Transaction Behavior

```python
# settings.py
DATABASES = {
    'default': {
        # ...
        'ATOMIC_REQUESTS': True,  # Wrap each request in transaction
    }
}
```

With `ATOMIC_REQUESTS=True`, if a view raises an exception, all database changes in that request are rolled back.\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test atomic transactions with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some atomic transactions features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- Atomic Transactions is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from Atomic Transactions**

```python
# WITHOUT transactions - DANGEROUS!
def transfer_money(from_account, to_account, amount):
    from_account.balance -= amount
    from_account.save()
    # What if server crashes here?
    to_account.balance += amount
    to_account.save()
```


## Resources

- [Database Transactions](https://docs.djangoproject.com/en/6.0/topics/db/transactions/) — Official guide to database transactions

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*