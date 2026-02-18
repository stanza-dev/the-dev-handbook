---
source_course: "django-deployment"
source_lesson: "django-deployment-requirements-management"
---

# Requirements and Dependencies

Properly managing dependencies ensures reproducible deployments.

## requirements.txt

```txt
# requirements/base.txt
Django>=6.0,<6.1
psycopg2-binary>=2.9
gunicorn>=21.0
whitenoise>=6.6
django-environ>=0.11

# requirements/production.txt
-r base.txt
sentry-sdk>=1.32
django-storages>=1.14
boto3>=1.28

# requirements/development.txt
-r base.txt
django-debug-toolbar>=4.2
pytest-django>=4.5
factory-boy>=3.3
```

## Poetry

```toml
# pyproject.toml
[tool.poetry]
name = "myproject"
version = "1.0.0"
description = "My Django Project"
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.11"
Django = "^6.0"
psycopg2-binary = "^2.9"
gunicorn = "^21.0"
whitenoise = "^6.6"

[tool.poetry.group.dev.dependencies]
pytest-django = "^4.5"
django-debug-toolbar = "^4.2"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

```bash
# Install dependencies
poetry install

# Add dependency
poetry add django-redis

# Export to requirements.txt
poetry export -f requirements.txt --output requirements.txt
```

## Pipenv

```
# Pipfile
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
django = ">=6.0,<6.1"
gunicorn = "*"
psycopg2-binary = "*"

[dev-packages]
pytest-django = "*"
django-debug-toolbar = "*"

[requires]
python_version = "3.11"
```

## Version Pinning Strategy

```txt
# Strict pinning for production stability
Django==6.0.2
gunicorn==21.2.0
whitenoise==6.6.0

# Using pip-compile (pip-tools)
# pip install pip-tools

# requirements.in
Django>=6.0,<6.1
gunicorn
whitenoise

# Generate pinned requirements
# pip-compile requirements.in > requirements.txt
```

## Security Updates

```bash
# Check for security vulnerabilities
pip install safety
safety check -r requirements.txt

# Or use pip-audit
pip install pip-audit
pip-audit

# Update packages
pip list --outdated
pip install --upgrade package-name
```

## Resources

- [Python Packaging](https://packaging.python.org/en/latest/guides/tool-recommendations/) — Python packaging tools guide

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*