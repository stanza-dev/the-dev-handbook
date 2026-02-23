---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-multi-db-migrations"
---

# Migration Strategies for Multi-DB Setups

## Introduction

Managing migrations across multiple databases requires controlling which models live on which database, running migrations against the right targets, and handling schema differences.

## Key Concepts

- **`allow_migrate()`**: Router method controlling whether a migration runs on a given database.
- **`--database` flag**: CLI argument targeting a specific database for migrate/showmigrations.
- **Unmanaged models**: Models with `managed = False` for external tables Django shouldn't alter.

## Real World Context

Production setups commonly have a primary DB, read replicas, an analytics warehouse, and legacy databases. Each needs a different migration strategy.

## Deep Dive

### Router-Controlled Migrations

Database routers control which models migrate to which database by implementing `allow_migrate()`. Return `True` to allow, `False` to deny, or `None` to delegate:

```python
class AnalyticsRouter:
    analytics_apps = {'analytics', 'reporting'}

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if app_label in self.analytics_apps:
            return db == 'analytics'
        if db == 'analytics':
            return False
        return None
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Running Migrations per Database

```bash
python manage.py migrate                      # Default DB
python manage.py migrate --database=analytics  # Analytics DB
python manage.py showmigrations --database=analytics
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Unmanaged Models

For tables owned by external systems or legacy applications, use `managed = False` to tell Django it should not create or alter the table:

```python
class LegacyCustomer(models.Model):
    customer_id = models.IntegerField(primary_key=True)
    full_name = models.CharField(max_length=200)

    class Meta:
        managed = False
        db_table = 'customers'

legacy = LegacyCustomer.objects.using('legacy').all()
```

## Common Pitfalls

1. **Forgetting `--database` flag** — `migrate` without it only touches the default database.
2. **`allow_migrate` returning None unexpectedly** — None delegates to the next router. Return True/False for definitive answers.
3. **Cross-database foreign keys** — Foreign keys across databases don't work. Use ID fields with manual lookups.

## Best Practices

1. **One router per concern** — Separate routers for replicas, analytics, and legacy databases.
2. **Use `managed = False` for external tables** — Never let Django manage tables owned by other systems.
3. **Automate in CI/CD** — Include all `--database` commands in deployment scripts.

## Summary

- Control migration targets with `allow_migrate()` in routers.
- Use `--database=<alias>` to target specific databases.
- Set `managed = False` for external/legacy tables.
- Automate multi-database migrations in your deployment pipeline.

## Code Examples

**Router controlling which migrations run on which database**

```python
class AnalyticsRouter:
    analytics_apps = {'analytics', 'reporting'}

    def allow_migrate(self, db, app_label, **hints):
        if app_label in self.analytics_apps:
            return db == 'analytics'
        if db == 'analytics':
            return False
        return None

# Run: python manage.py migrate --database=analytics
```


## Resources

- [Multiple Databases](https://docs.djangoproject.com/en/6.0/topics/db/multi-db/) — Official guide to multi-database setups

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*