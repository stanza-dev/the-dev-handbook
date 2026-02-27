---
source_course: "python"
source_lesson: "python-standard-library"
---

# Essential Standard Library Modules

## Introduction
Python's standard library is one of its greatest strengths -- often called "batteries included." This lesson tours the modules you will reach for most often: `os`/`shutil` for system operations, `json` for data serialization, `datetime` for time handling, `logging` for diagnostics, and `argparse` for CLI tools.

## Key Concepts
- **`os` / `shutil`**: System-level file and directory operations.
- **`json`**: Serialize Python objects to JSON strings and parse them back.
- **`datetime`**: Work with dates, times, and durations.
- **`logging`**: Structured, leveled diagnostic output (INFO, WARNING, ERROR).
- **`argparse`**: Parse command-line arguments into a typed namespace.

## Real World Context
These modules appear in virtually every Python project. Web APIs use `json` to parse request bodies, deployment scripts use `shutil` to copy artifacts, backend services use `logging` instead of `print()` for production diagnostics, and CLI tools use `argparse` to provide a polished user interface. Knowing them well means you rarely need third-party packages for basic tasks.

## Deep Dive

### os and shutil -- System Operations

```python
import os
import shutil

os.getcwd()              # Current directory
os.listdir('.')          # List directory
os.environ['PATH']       # Environment variables
os.makedirs('a/b/c', exist_ok=True)

shutil.copy('src', 'dst')      # Copy file
shutil.copytree('src', 'dst')  # Copy directory
shutil.rmtree('dir')           # Remove directory tree
```

### json -- Data Serialization

```python
import json

# Serialize
data = {'name': 'Alice', 'age': 30}
json_str = json.dumps(data, indent=2)

# Parse
data = json.loads(json_str)

# File I/O
with open('data.json', 'w') as f:
    json.dump(data, f)

with open('data.json') as f:
    data = json.load(f)
```

### datetime -- Date and Time

```python
from datetime import datetime, date, timedelta

now = datetime.now()
today = date.today()

# Formatting
now.strftime('%Y-%m-%d %H:%M:%S')  # '2024-01-15 14:30:00'

# Parsing
datetime.strptime('2024-01-15', '%Y-%m-%d')

# Arithmetic
tomorrow = today + timedelta(days=1)
```

### logging -- Application Logging

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)
logger.info('Application started')
logger.warning('Low memory')
logger.error('Connection failed')
```

### argparse -- CLI Arguments

```python
import argparse

parser = argparse.ArgumentParser(description='My CLI tool')
parser.add_argument('filename', help='Input file')
parser.add_argument('-v', '--verbose', action='store_true')
parser.add_argument('-n', '--count', type=int, default=1)

args = parser.parse_args()
print(args.filename, args.verbose, args.count)
```

## Common Pitfalls
1. **Using `print()` instead of `logging` in production** -- `print()` goes to stdout with no timestamp, level, or source. Switch to `logging` for any code that runs in production.
2. **Using naive datetimes across time zones** -- `datetime.now()` returns a naive (timezone-unaware) object. Use `datetime.now(timezone.utc)` to avoid bugs when your code runs in different time zones.
3. **Serializing non-JSON types with `json.dumps`** -- Passing a `datetime` or `set` to `json.dumps` raises `TypeError`. Use the `default` parameter or convert to a serializable type first.

## Best Practices
1. **Use `pathlib` alongside `os`/`shutil`** -- Pathlib handles path construction; shutil handles higher-level operations like `copytree` and `rmtree`.
2. **Create a logger per module with `logging.getLogger(__name__)`** -- This gives you hierarchical, filterable log output without polluting the root logger.

## Summary
- `os` and `shutil` handle system-level file operations; combine them with `pathlib` for clean path handling.
- `json` serializes and parses data; always handle non-serializable types with `default`.
- `datetime` handles dates and durations; always use timezone-aware objects in production.
- `logging` replaces `print()` with structured, leveled output for production diagnostics.
- `argparse` builds professional CLI interfaces with typed arguments and help text.

## Code Examples

**Standard library overview**

```python
# Useful stdlib modules overview
import re           # Regular expressions
import hashlib      # Cryptographic hashes
import secrets      # Secure random numbers
import urllib       # URL handling
import sqlite3      # SQLite database
import subprocess   # Run external commands
import threading    # Threads
import typing       # Type hints
import dataclasses  # Data classes
import functools    # Function tools (lru_cache, partial)
```


## Resources

- [Python Standard Library](https://docs.python.org/3.14/library/index.html) — Official Python 3.14 standard library reference index

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*