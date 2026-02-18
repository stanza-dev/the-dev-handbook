---
source_course: "django-foundations"
source_lesson: "django-foundations-creating-first-project"
---

# Creating Your First Django Project

Let's create your first Django project and explore its structure. This hands-on experience will help you understand how Django organizes code.

## Creating a New Project

With your virtual environment activated, run:

```bash
# Create a new Django project called 'mysite'
django-admin startproject mysite

# Navigate into the project directory
cd mysite
```

Django generates this structure:

```
mysite/                 # Root directory (can be renamed)
├── manage.py           # Command-line utility
└── mysite/             # Project package
    ├── __init__.py     # Python package marker
    ├── settings.py     # Project configuration
    ├── urls.py         # URL routing
    ├── asgi.py         # ASGI entry point
    └── wsgi.py         # WSGI entry point
```

## Understanding Key Files

### manage.py

A command-line utility for interacting with your project:

```bash
# Run the development server
python manage.py runserver

# Create database migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create a superuser
python manage.py createsuperuser

# Open Django shell
python manage.py shell
```

### settings.py

Contains all project configuration:

```python
# mysite/settings.py

# SECURITY WARNING: keep the secret key secret in production!
SECRET_KEY = 'django-insecure-...'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# Database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

### urls.py

Defines how URLs map to views:

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]
```

## Starting the Development Server

Django includes a lightweight development server:

```bash
python manage.py runserver
```

You'll see output like:

```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s)...
Run 'python manage.py migrate' to apply them.

Django version 6.0, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Open **http://127.0.0.1:8000** in your browser. You should see the Django welcome page with a rocket!

## Run Initial Migrations

Django comes with built-in apps that need database tables:

```bash
python manage.py migrate
```

Output:

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  ...
```

This creates the SQLite database file `db.sqlite3` with tables for authentication, sessions, and more.

## Resources

- [Writing your first Django app, part 1](https://docs.djangoproject.com/en/6.0/intro/tutorial01/) — Official tutorial for creating your first Django project

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*