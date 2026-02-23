---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-multiple-db-config"
---

# Configuring Multiple Databases

Django supports multiple database connections, allowing you to spread data across different database servers.

## Database Configuration

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'primary_db',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'primary.db.server',
        'PORT': '5432',
    },
    'replica': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'primary_db',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'replica.db.server',
        'PORT': '5432',
    },
    'analytics': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'analytics_db',
        'USER': 'analytics',
        'PASSWORD': 'password',
        'HOST': 'analytics.db.server',
        'PORT': '5432',
    },
    'legacy': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'legacy_app',
        'USER': 'root',
        'PASSWORD': 'password',
        'HOST': 'legacy.db.server',
        'PORT': '3306',
    },
}
```

## Using a Specific Database

### With QuerySets

The following example demonstrates how to use with querysets in practice:

```python
# Read from replica
users = User.objects.using('replica').all()

# Write to default
new_user = User.objects.using('default').create(
    username='john',
    email='john@example.com'
)

# Query analytics database
reports = AnalyticsReport.objects.using('analytics').filter(
    date__gte=last_month
)
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### With Model Instances

The following example demonstrates how to use with model instances in practice:

```python
# Save to specific database
user = User(username='jane', email='jane@example.com')
user.save(using='default')

# Delete from specific database
user.delete(using='default')

# Refresh from specific database
user.refresh_from_db(using='replica')
```

## Database Routers

Automate database selection with routers:

```python
# routers.py
class PrimaryReplicaRouter:
    """
    Route reads to replica, writes to primary.
    """
    
    def db_for_read(self, model, **hints):
        """Direct reads to replica."""
        return 'replica'
    
    def db_for_write(self, model, **hints):
        """Direct writes to primary."""
        return 'default'
    
    def allow_relation(self, obj1, obj2, **hints):
        """Allow relations between objects in same db cluster."""
        db_set = {'default', 'replica'}
        if obj1._state.db in db_set and obj2._state.db in db_set:
            return True
        return None
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """Only migrate on primary."""
        return db == 'default'
```

```python
# settings.py
DATABASE_ROUTERS = ['myapp.routers.PrimaryReplicaRouter']
```

## App-Specific Routing

```python
class AppRouter:
    """
    Route different apps to different databases.
    """
    
    route_app_labels = {'analytics', 'reporting'}
    
    def db_for_read(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return 'analytics'
        return 'default'
    
    def db_for_write(self, model, **hints):
        if model._meta.app_label in self.route_app_labels:
            return 'analytics'
        return 'default'
    
    def allow_relation(self, obj1, obj2, **hints):
        # Only allow relations within the same database
        if (
            obj1._meta.app_label in self.route_app_labels or
            obj2._meta.app_label in self.route_app_labels
        ):
            return obj1._meta.app_label == obj2._meta.app_label
        return None
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if app_label in self.route_app_labels:
            return db == 'analytics'
        return db == 'default'
```

## Running Migrations

```bash
# Migrate default database
python manage.py migrate

# Migrate specific database
python manage.py migrate --database=analytics

# Migrate all databases
python manage.py migrate --database=default
python manage.py migrate --database=analytics
python manage.py migrate --database=legacy
```

## Cross-Database Queries

```python
# Cross-database queries are NOT supported by Django ORM
# You must handle this manually

# Get IDs from analytics
report_user_ids = list(
    AnalyticsEvent.objects.using('analytics')
    .values_list('user_id', flat=True)
    .distinct()
)

# Query main database with those IDs
active_users = User.objects.using('default').filter(
    id__in=report_user_ids
)
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test configuring multiple databases with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some configuring multiple databases features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- Configuring Multiple Databases is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from Configuring Multiple Databases**

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'primary_db',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'primary.db.server',
        'PORT': '5432',
    },
    'replica': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'primary_db',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'replica.db.server',
        'PORT': '5432',
    },
    'analytics': {
 
```


## Resources

- [Multiple Databases](https://docs.djangoproject.com/en/6.0/topics/db/multi-db/) — Official guide to multiple databases

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*