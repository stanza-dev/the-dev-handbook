---
source_course: "python-architecture"
source_lesson: "python-architecture-package-structure"
---

# Package Structure & Namespaces

## Introduction

As Python projects grow beyond a single file, organizing code into packages becomes essential. A package is a directory containing an `__init__.py` file (regular package) or no `__init__.py` at all (namespace package). Understanding how packages, relative imports, and re-exporting work is critical for building maintainable APIs.

## Key Concepts

- **Regular package**: A directory with an `__init__.py` file that Python treats as a package.
- **`__init__.py`**: Executed when the package is imported; controls what the package exposes.
- **Relative imports**: Imports using dots (`.`) to reference sibling or parent modules within the same package.
- **Namespace package**: A package without `__init__.py` (PEP 420), allowing a single logical package to span multiple directories.
- **Re-exporting**: Importing names in `__init__.py` so consumers can access them from the package root.

## Real World Context

Consider the `requests` library structure: users write `from requests import get, Session` even though `get` lives in `requests.api` and `Session` lives in `requests.sessions`. This is possible because `requests/__init__.py` re-exports these names, providing a clean, flat public API while keeping the internal structure organized.

## Deep Dive

### Basic package structure

```
mylib/
├── __init__.py        # Package entry point
├── models.py          # Data models
├── services.py        # Business logic
└── _helpers.py        # Internal utilities
```

```python
# mylib/__init__.py
"""MyLib - a clean public API."""

from .models import User, Product
from .services import create_user, fetch_products

__all__ = ['User', 'Product', 'create_user', 'fetch_products']
```

### Relative imports

```python
# mylib/services.py
from .models import User          # Same-level import
from ._helpers import validate     # Same-level private module
from ..config import settings      # Parent package import

def create_user(name: str) -> User:
    validate(name)
    return User(name=name)
```

### Nested package structure

```
mylib/
├── __init__.py
├── api/
│   ├── __init__.py
│   ├── v1.py
│   └── v2.py
├── core/
│   ├── __init__.py
│   └── engine.py
└── utils/
    ├── __init__.py
    └── formatting.py
```

```python
# mylib/api/__init__.py
from .v2 import router  # Default to latest version

# mylib/core/engine.py
from ..utils.formatting import format_output  # Cross-package relative import
```

### Namespace packages (PEP 420)

Namespace packages allow a single package to be split across multiple directories or distributions:

```
# Directory A (installed by package-a)
mynamespace/
    subpackage_a/
        __init__.py
        module_a.py

# Directory B (installed by package-b)
mynamespace/
    subpackage_b/
        __init__.py
        module_b.py
```

```python
# Both work because mynamespace is a namespace package (no __init__.py)
import mynamespace.subpackage_a
import mynamespace.subpackage_b
```

### Re-exporting for clean APIs

```python
# mylib/__init__.py
# Re-export from submodules so users write:
#   from mylib import User, create_user
# instead of:
#   from mylib.models import User
#   from mylib.services import create_user

from .models import User, Product
from .services import create_user

__all__ = ['User', 'Product', 'create_user']
```

## Common Pitfalls

1. **Circular imports**: Module A imports from Module B, which imports from Module A. Fix by moving shared types to a separate module or using local imports.
2. **Missing `__init__.py`**: Without it, Python 3 treats the directory as a namespace package, which may cause unexpected import behavior.
3. **Relative import outside package**: Running `python mylib/services.py` directly fails with relative imports. Use `python -m mylib.services` instead.

## Best Practices

- Keep `__init__.py` files small — they should primarily re-export, not contain business logic.
- Use relative imports within a package for portability (the package can be renamed without changing internal imports).
- Combine re-exporting with `__all__` to create a curated, flat public API.
- Avoid deep nesting — if your imports look like `from mylib.core.utils.internal.helpers import x`, consider restructuring.

## Summary

Python packages are directories with `__init__.py` files that organize code into logical units. Relative imports keep packages self-contained, re-exporting in `__init__.py` creates clean public APIs, and namespace packages enable multi-distribution packages. Together, these features let you build well-structured libraries with clear boundaries between public and private code.

## Code Examples

**Re-exporting names for a clean package API**

```python
# project/payments/__init__.py
"""Payments package - clean public API."""

from .processor import process_payment, refund
from .models import PaymentResult, PaymentStatus
from ._gateway import _StripeGateway  # noqa: F401 (internal)

__all__ = [
    'process_payment',
    'refund',
    'PaymentResult',
    'PaymentStatus',
]

# Consumer code:
# from payments import process_payment, PaymentResult
# result: PaymentResult = process_payment(amount=99.99)
```

**Relative imports within a package**

```python
# project/payments/processor.py
from .models import PaymentResult, PaymentStatus
from ._gateway import _StripeGateway
from ..config import settings  # Parent package

def process_payment(amount: float) -> PaymentResult:
    gateway = _StripeGateway(api_key=settings.STRIPE_KEY)
    success = gateway.charge(amount)
    return PaymentResult(
        status=PaymentStatus.SUCCESS if success else PaymentStatus.FAILED,
        amount=amount,
    )
```


## Resources

- [Python Packages](https://docs.python.org/3/tutorial/modules.html#packages) — Official Python tutorial on packages and imports
- [PEP 328 – Imports: Multi-Line and Absolute/Relative](https://peps.python.org/pep-0328/) — PEP introducing relative imports

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*