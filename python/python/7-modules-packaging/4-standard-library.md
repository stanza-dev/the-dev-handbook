---
source_course: "python"
source_lesson: "python-standard-library"
---

# Essential Standard Library

## os and shutil - System Operations

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

## json - Data Serialization

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

## datetime - Date and Time

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

## logging - Application Logging

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

## argparse - CLI Arguments

```python
import argparse

parser = argparse.ArgumentParser(description='My CLI tool')
parser.add_argument('filename', help='Input file')
parser.add_argument('-v', '--verbose', action='store_true')
parser.add_argument('-n', '--count', type=int, default=1)

args = parser.parse_args()
print(args.filename, args.verbose, args.count)
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*