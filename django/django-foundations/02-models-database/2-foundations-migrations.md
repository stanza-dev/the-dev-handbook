---
source_course: "django-foundations"
source_lesson: "django-foundations-migrations"
---

# Database Migrations

Migrations are Django's way of propagating changes you make to your models into your database schema. Think of them as version control for your database.

## Why Migrations?

- **Track changes**: Every schema change is recorded
- **Team collaboration**: Share database changes with your team
- **Database agnostic**: Same migrations work on SQLite, PostgreSQL, MySQL
- **Reversible**: Roll back changes if needed

## The Migration Workflow

```
1. Edit models.py     2. makemigrations      3. migrate
┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐
│ class Question: │   │ 0001_initial.py  │   │   DATABASE      │
│   question_text │ → │ CreateModel      │ → │   questions     │
│   pub_date      │   │   Question       │   │   table created │
└─────────────────┘   └──────────────────┘   └─────────────────┘
```

## Step 1: Create Migrations

After defining or modifying your models:

```bash
python manage.py makemigrations polls
```

Output:

```
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```

Django creates a migration file:

```python
# polls/migrations/0001_initial.py
from django.db import migrations, models
import django.db.models.deletion


class Migration(migrations.Migration):

    initial = True

    dependencies = []

    operations = [
        migrations.CreateModel(
            name='Question',
            fields=[
                ('id', models.BigAutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('question_text', models.CharField(max_length=200)),
                ('pub_date', models.DateTimeField(verbose_name='date published')),
            ],
        ),
        migrations.CreateModel(
            name='Choice',
            fields=[
                ('id', models.BigAutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
                ('choice_text', models.CharField(max_length=200)),
                ('votes', models.IntegerField(default=0)),
                ('question', models.ForeignKey(on_delete=django.db.models.deletion.CASCADE, to='polls.question')),
            ],
        ),
    ]
```

## Step 2: Preview the SQL

See what SQL Django will run:

```bash
python manage.py sqlmigrate polls 0001
```

Output:

```sql
CREATE TABLE "polls_question" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "question_text" varchar(200) NOT NULL,
    "pub_date" datetime NOT NULL
);
CREATE TABLE "polls_choice" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" bigint NOT NULL REFERENCES "polls_question" ("id")
);
```

## Step 3: Apply Migrations

Run the migration to create the tables:

```bash
python manage.py migrate
```

Output:

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

## Common Migration Commands

Django provides several migration commands for different stages of the workflow. Here is a reference of the most useful ones.

```bash
# Create migrations for all apps
python manage.py makemigrations

# Create migrations for specific app
python manage.py makemigrations polls

# Apply all pending migrations
python manage.py migrate

# Apply migrations for specific app
python manage.py migrate polls

# See migration status
python manage.py showmigrations

# Roll back to a specific migration
python manage.py migrate polls 0001

# Roll back all migrations for an app
python manage.py migrate polls zero
```

## Adding Fields Later

When you add new fields to existing models:

```python
# polls/models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    is_active = models.BooleanField(default=True)  # New field
```

Run migrations:

```bash
python manage.py makemigrations polls
python manage.py migrate
```

Django asks how to handle existing rows:

```
You are trying to add a non-nullable field 'is_active' to question...
 1) Provide a one-off default now
 2) Quit, and let me add a default in models.py
```

Using `default=True` avoids this prompt.

## Common Pitfalls

- **Editing migration files manually**: Migration files are auto-generated. Manual edits can corrupt your migration history and cause hard-to-debug errors.
- **Forgetting to commit migrations to version control**: Migration files must be shared with your team. Missing migrations cause deployment failures.
- **Adding non-nullable fields without defaults**: When adding a new required field to an existing model, always provide a `default` value or Django will prompt you interactively.

## Best Practices

- **Run `makemigrations` before `migrate`**: Always generate migration files first, then apply them. Never skip `makemigrations`.
- **Review migration files before applying**: Check what SQL Django will run with `python manage.py sqlmigrate app_name migration_number`.
- **Use `showmigrations` to check status**: Run `python manage.py showmigrations` to see which migrations have been applied.

## Summary

- Migrations are Django's **version control for database schemas**
- The workflow is: edit `models.py` -> `makemigrations` -> `migrate`
- `makemigrations` creates migration files describing schema changes
- `migrate` applies those changes to the database
- Use `showmigrations` to check migration status and `sqlmigrate` to preview SQL

## Code Examples

**The standard Django migration workflow: edit models, create migrations, apply them**

```bash
# The three-step migration workflow
# 1. Edit your models in models.py
# 2. Create the migration file
python manage.py makemigrations polls
# 3. Apply the migration to the database
python manage.py migrate
```


## Resources

- [Django Migrations](https://docs.djangoproject.com/en/6.0/topics/migrations/) — Official guide to database migrations

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*