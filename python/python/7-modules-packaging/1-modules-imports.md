---
source_course: "python"
source_lesson: "python-modules-imports"
---

# Python Modules

A module is a file containing Python code. A package is a directory containing modules and an `__init__.py` file.

## Import Syntax

```python
# Import entire module
import math
print(math.sqrt(16))

# Import specific items
from math import sqrt, pi
print(sqrt(16))

# Import with alias
import numpy as np
from collections import defaultdict as dd

# Import all (avoid in production)
from math import *
```

## Module Search Path

Python searches for modules in this order:
1. Built-in modules
2. Current directory
3. `PYTHONPATH` environment variable
4. Site-packages (installed packages)

```python
import sys
print(sys.path)  # List of search directories
```

## The `__name__` Variable

```python
# mymodule.py
def main():
    print("Running as main")

if __name__ == "__main__":
    main()  # Only runs when executed directly
```

## Relative vs Absolute Imports

```python
# Inside a package:
# Absolute import
from mypackage.utils import helper

# Relative import
from .utils import helper      # Same package
from ..other import something  # Parent package
```

## Package `__init__.py`

```python
# mypackage/__init__.py
from .core import MainClass
from .utils import helper

__all__ = ['MainClass', 'helper']  # Controls * imports
__version__ = '1.0.0'
```

## Code Examples

**Import patterns**

```python
# Lazy imports for performance
def process_data():
    import pandas as pd  # Only imported when function is called
    return pd.DataFrame()

# Conditional imports
try:
    import ujson as json  # Fast JSON library
except ImportError:
    import json  # Fallback to standard library
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*