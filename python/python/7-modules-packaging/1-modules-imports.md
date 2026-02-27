---
source_course: "python"
source_lesson: "python-modules-imports"
---

# Modules and Imports

## Introduction
Modules and packages are how Python organizes code into reusable, maintainable units. This lesson covers import syntax, the module search path, the `__name__` guard, relative vs absolute imports, and how `__init__.py` defines a package's public API.

## Key Concepts
- **Module**: A single `.py` file containing Python code.
- **Package**: A directory containing modules and an `__init__.py` file.
- **`import` / `from ... import`**: Statements that load modules or specific names from modules.
- **`__name__`**: A special variable that equals `"__main__"` when a file is run directly.
- **Relative import**: Uses dots (`.`, `..`) to import from within the same package.

## Real World Context
Every Python project larger than a script relies on a well-organized module structure. Django apps, Flask blueprints, and data science packages all use `__init__.py` to control their public API and relative imports to avoid hardcoded package names. Understanding the import system is also critical for resolving the "ModuleNotFoundError" that blocks beginners regularly.

## Deep Dive

### Import Syntax

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

### Module Search Path

Python searches for modules in this order:
1. Built-in modules
2. Current directory
3. `PYTHONPATH` environment variable
4. Site-packages (installed packages)

```python
import sys
print(sys.path)  # List of search directories
```

### The `__name__` Variable

```python
# mymodule.py
def main():
    print("Running as main")

if __name__ == "__main__":
    main()  # Only runs when executed directly
```

### Relative vs Absolute Imports

```python
# Inside a package:
# Absolute import
from mypackage.utils import helper

# Relative import
from .utils import helper      # Same package
from ..other import something  # Parent package
```

### Package `__init__.py`

```python
# mypackage/__init__.py
from .core import MainClass
from .utils import helper

__all__ = ['MainClass', 'helper']  # Controls * imports
__version__ = '1.0.0'
```

## Common Pitfalls
1. **Circular imports** -- Module A imports from Module B, which imports from Module A. This causes `ImportError` or `None` attributes. Fix by moving shared code to a third module or using lazy imports inside functions.
2. **Using `from module import *` in production** -- It pollutes the namespace and makes it impossible to know where a name came from. Always import specific names.
3. **Running a module that uses relative imports directly** -- Relative imports only work within a package. Running `python mypackage/utils.py` directly will fail. Use `python -m mypackage.utils` instead.

## Best Practices
1. **Use absolute imports in most cases** -- They are explicit, unambiguous, and work regardless of how the script is invoked.
2. **Define `__all__` in `__init__.py`** -- It documents the public API and controls what `from package import *` exports.

## Summary
- Modules are `.py` files; packages are directories with `__init__.py`.
- Python searches built-in modules, then the current directory, `PYTHONPATH`, and site-packages.
- The `if __name__ == "__main__":` guard runs code only when the file is executed directly.
- Use absolute imports for clarity; use relative imports within packages.
- Avoid `from module import *` and define `__all__` to control your package's public API.

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


## Resources

- [Modules](https://docs.python.org/3.14/tutorial/modules.html) — Official Python 3.14 tutorial on modules, imports, and the module search path

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*