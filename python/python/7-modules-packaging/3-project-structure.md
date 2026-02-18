---
source_course: "python"
source_lesson: "python-project-structure"
---

# Modern Python Project Structure

## Recommended Layout

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

## pyproject.toml

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

## Virtual Environments

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

## Environment Variables

```python
import os
from dotenv import load_dotenv

load_dotenv()  # Load from .env file

DATABASE_URL = os.environ.get('DATABASE_URL', 'sqlite:///db.sqlite')
DEBUG = os.environ.get('DEBUG', 'false').lower() == 'true'
```

## Code Examples

**Tool configuration in pyproject.toml**

```toml
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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*