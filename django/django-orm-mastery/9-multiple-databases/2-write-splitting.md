---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-read-write-splitting"
---

# Read/Write Splitting

Directing read queries to replicas reduces load on the primary database and improves performance.

## Basic Router

```python
import random

class ReadWriteRouter:
    """
    Route reads to replicas, writes to primary.
    """
    
    read_replicas = ['replica1', 'replica2', 'replica3']
    
    def db_for_read(self, model, **hints):
        # Random replica selection
        return random.choice(self.read_replicas)
    
    def db_for_write(self, model, **hints):
        return 'default'
    
    def allow_relation(self, obj1, obj2, **hints):
        return True
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        return db == 'default'
```

## Handling Replication Lag

Replicas may be slightly behind the primary:

```python
class SmartRouter:
    """
    Smart routing with replication lag awareness.
    """
    
    def db_for_read(self, model, **hints):
        # If we just wrote, read from primary to avoid lag issues
        if hints.get('instance'):
            instance = hints['instance']
            if hasattr(instance, '_just_saved'):
                return 'default'
        
        return 'replica'
    
    def db_for_write(self, model, **hints):
        return 'default'
```

## Forcing Primary Reads

```python
# When you need fresh data
user = User.objects.using('default').get(pk=user_id)  # Force primary

# Or with a context manager
from django.db import router

class ForcePrimary:
    """Context manager to force reads from primary."""
    
    def __enter__(self):
        self._original = router.routers
        router.routers = []  # Disable routing
        return self
    
    def __exit__(self, *args):
        router.routers = self._original

# Usage
with ForcePrimary():
    user = User.objects.get(pk=user_id)  # Always hits default
```

## Transactions Across Databases

```python
from django.db import transaction

# Transaction on specific database
with transaction.atomic(using='analytics'):
    AnalyticsEvent.objects.using('analytics').create(...)

# Multi-database atomic is NOT supported
# Each database has its own transaction

def transfer_data():
    # These are SEPARATE transactions
    with transaction.atomic(using='default'):
        source.delete()  # default db
    
    with transaction.atomic(using='analytics'):
        AnalyticsRecord.objects.create(...)  # analytics db
    
    # If second fails, first is already committed!
```

## Health-Aware Routing

```python
import time
from django.db import connections


class HealthAwareRouter:
    """
    Check replica health before routing.
    """
    
    _replica_status = {}  # Cache status
    _check_interval = 60  # Seconds
    
    def _is_healthy(self, db_alias):
        now = time.time()
        last_check = self._replica_status.get(db_alias, {}).get('checked', 0)
        
        if now - last_check > self._check_interval:
            try:
                conn = connections[db_alias]
                conn.ensure_connection()
                healthy = True
            except Exception:
                healthy = False
            
            self._replica_status[db_alias] = {
                'healthy': healthy,
                'checked': now
            }
        
        return self._replica_status.get(db_alias, {}).get('healthy', False)
    
    def db_for_read(self, model, **hints):
        for replica in ['replica1', 'replica2']:
            if self._is_healthy(replica):
                return replica
        return 'default'  # Fallback to primary
```

## Practical Configuration

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myapp',
        'HOST': 'primary.db.server',
        'CONN_MAX_AGE': 600,
    },
    'replica': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myapp',
        'HOST': 'replica.db.server',
        'CONN_MAX_AGE': 600,
        'TEST': {
            'MIRROR': 'default',  # Don't create separate test db
        },
    },
}

DATABASE_ROUTERS = ['myapp.routers.ReadWriteRouter']
```

## Resources

- [Database Routing](https://docs.djangoproject.com/en/6.0/topics/db/multi-db/#automatic-database-routing) — Automatic database routing reference

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*