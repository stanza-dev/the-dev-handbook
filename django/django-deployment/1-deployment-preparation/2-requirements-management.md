---
source_course: "django-deployment"
source_lesson: "django-deployment-requirements-management"
---

# Requirements and Dependencies

## Introduction

Properly managing Python dependencies ensures reproducible deployments and prevents "works on my machine" issues. Modern Python offers several tools for dependency management, each with trade-offs around lockfile support, virtual environment integration, and ecosystem compatibility.

## Key Concepts

- **requirements.txt**: The traditional pip-based format for listing dependencies with optional version pinning.
- **Poetry**: A modern dependency manager with lockfile support, virtual environment management, and pyproject.toml-based configuration.
- **pip-tools**: A lightweight tool that compiles loose requirements into fully pinned lockfiles.
- **Version Pinning**: Specifying exact package versions to ensure identical installations across environments.

## Real World Context

Unpinned dependencies are a common source of deployment failures. A library releasing a breaking change at 2 AM on a Friday has caused countless production outages. Teams that pin their dependencies and use lockfiles (Poetry's poetry.lock, pip-tools' requirements.txt) can deploy with confidence knowing the exact same package versions run in every environment.

## Deep Dive

Reproducible deployments require pinned dependencies. Modern Python tooling like Poetry or pip-tools generates lock files that ensure identical installs across environments.


## Introduction

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

```python
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

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Not pinning dependencies** — Using `Django>=6.0` without an upper bound means a future breaking release could be installed automatically during deployment.
2. **Mixing dependency managers** — Using both Poetry and pip requirements.txt creates confusion about which is the source of truth.
3. **Ignoring security advisories** — Regularly run `pip-audit` or `safety check` to detect known vulnerabilities in your dependency tree.

## Best Practices

1. **Use a lockfile** — Whether Poetry's poetry.lock or pip-compile's output, always deploy with pinned, deterministic dependency versions.
2. **Separate dev and production dependencies** — Keep testing and debugging tools out of production to reduce attack surface and image size.
3. **Automate vulnerability scanning** — Add `pip-audit` to your CI pipeline to catch vulnerable packages before they reach production.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Poetry pyproject.toml for managing Django project dependencies**

```python
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
# ...
```


## Resources

- [Python Packaging](https://packaging.python.org/en/latest/guides/tool-recommendations/) — Python packaging tools guide

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*