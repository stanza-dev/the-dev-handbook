---
source_course: "python"
source_lesson: "python-project-structure"
---

# Modern Project Structure

## Introduction
A well-organized project structure is the difference between a hobby script and a maintainable, deployable application. This lesson covers the recommended `src` layout, `pyproject.toml` for metadata and dependencies, virtual environments, and environment variable management.

## Key Concepts
- **src layout**: A project structure where package code lives under `src/`, isolating it from tests and configuration.
- **`pyproject.toml`**: The modern standard (PEP 518, PEP 621) for declaring project metadata, dependencies, and tool configuration in one file.
- **Virtual environment**: An isolated Python installation that keeps project dependencies separate from the system.
- **Environment variables**: Configuration values (database URLs, API keys) loaded at runtime, never hardcoded.

## Real World Context
Every open-source Python package on PyPI uses `pyproject.toml` and a virtual environment. CI/CD pipelines create virtual environments to run tests in isolation, and deployment scripts read environment variables for secrets. Getting the project structure right from the start saves hours of refactoring later.

## Deep Dive

### Recommended Layout

```
my_project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── core.py
│       └── utils.py
├── tests/
│   ├── __init__.py
│   └── test_core.py
├── docs/
├── pyproject.toml
├── README.md
└── .gitignore
```

### pyproject.toml

The modern standard for Python project configuration (PEP 518, PEP 621):

```toml
[project]
name = "my-project"
version = "1.0.0"
description = "A great Python project"
authors = [
    {name = "Your Name", email = "you@example.com"}
]
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28.0",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "black>=23.0",
]

[project.scripts]
my-cli = "my_project.cli:main"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

### Virtual Environments

```bash
# Create virtual environment
python -m venv .venv

# Activate (Unix/macOS)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install dependencies
pip install -e ".[dev]"  # Editable install with dev deps
```

### Environment Variables

```python
import os
from dotenv import load_dotenv

load_dotenv()  # Load from .env file

DATABASE_URL = os.environ.get('DATABASE_URL', 'sqlite:///db.sqlite')
DEBUG = os.environ.get('DEBUG', 'false').lower() == 'true'
```

## Common Pitfalls
1. **Putting package code at the project root without `src/`** -- Tests can accidentally import from the local directory instead of the installed package, masking import errors that would appear in production.
2. **Committing `.env` files to version control** -- Environment files often contain secrets. Add `.env` to `.gitignore` and use `.env.example` as a template.
3. **Installing packages globally instead of in a virtual environment** -- Global installs cause dependency conflicts between projects. Always use `python -m venv .venv`.

## Best Practices
1. **Use `pyproject.toml` for all configuration** -- Consolidate tool settings (black, pytest, mypy, ruff) in one file instead of scattering them across `setup.cfg`, `tox.ini`, and `.flake8`.
2. **Use editable installs during development** -- `pip install -e ".[dev]"` lets you import your package without reinstalling after every change.

## Summary
- The `src/` layout separates package code from tests and configuration, preventing accidental local imports.
- `pyproject.toml` is the single source of truth for metadata, dependencies, and tool configuration.
- Virtual environments isolate project dependencies; always create one per project.
- Load secrets from environment variables, never hardcode them.
- Use `pip install -e ".[dev]"` for a development workflow that auto-reflects code changes.

## Code Examples

**Tool configuration in pyproject.toml**

```yaml
# Modern pyproject.toml with tools config
[project]
name = "my-project"
version = "1.0.0"

[tool.black]
line-length = 88

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=src"

[tool.mypy]
strict = true

[tool.ruff]
select = ["E", "F", "I"]
ignore = ["E501"]
```


## Resources

- [Packages](https://docs.python.org/3.14/tutorial/modules.html#packages) — Official Python 3.14 tutorial on packages, __init__.py, and project organization

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*