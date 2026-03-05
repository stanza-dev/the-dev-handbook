---
source_course: "python-performance"
source_lesson: "python-performance-pyo3"
---

# PyO3: Rust Extensions

## Introduction

Rust has emerged as the preferred language for new Python extensions thanks to its unique combination of C-level performance, compile-time memory safety, and excellent Python bindings via the PyO3 crate. With the `maturin` build tool, creating and distributing Rust-based Python packages is remarkably straightforward.

## Key Concepts

- **PyO3**: A Rust crate (library) that provides bindings between Rust and Python, handling reference counting, type conversions, and GIL management automatically.
- **maturin**: A build tool that compiles Rust code into Python-compatible wheel packages, handling cross-compilation and distribution for pip install.
- **Memory safety**: Rust's ownership system prevents null pointer dereferences, buffer overflows, data races, and use-after-free bugs at compile time, eliminating entire categories of extension crashes.
- **GIL release**: PyO3 makes it easy to release the GIL during CPU-intensive Rust code, enabling true multi-threaded parallelism from Python.

## Real World Context

The Rust-in-Python ecosystem has exploded: pydantic v2 achieved 5-50x validation speedups by rewriting its core in Rust, ruff replaced flake8 + isort + pyupgrade with a single tool that's 10-100x faster, and polars provides a pandas alternative that handles DataFrames orders of magnitude faster. These projects demonstrate that Rust extensions can dramatically improve Python tool performance while maintaining a pure Python API.

## Deep Dive

### Why Rust for Python Extensions?

| Feature | C Extension | Rust (PyO3) |
|---------|------------|-------------|
| Memory safety | Manual (segfaults) | Compile-time guaranteed |
| Thread safety | Manual (data races) | Compile-time guaranteed |
| Performance | Maximum | Maximum (zero-cost abstractions) |
| Build system | Complex (setuptools) | Simple (maturin) |
| Error handling | Error codes | Result types (no crashes) |
| Learning curve | Steep + dangerous | Steep but safe |

### Basic PyO3 Example

Here is a simple Rust function exposed to Python:

```rust
// src/lib.rs
use pyo3::prelude::*;

/// Calculate the sum of squares from 0 to n
#[pyfunction]
fn sum_squares(n: u64) -> u64 {
    (0..n).map(|i| i * i).sum()
}

/// A Python module implemented in Rust
#[pymodule]
fn fast_math(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_squares, m)?)?;
    Ok(())
}
```

The `#[pyfunction]` attribute makes a Rust function callable from Python, and `#[pymodule]` creates the module entry point.

### Building with maturin

```bash
# Install maturin
pip install maturin

# Create a new project
maturin new my_extension --bindings pyo3
cd my_extension

# Build and install in development mode
maturin develop

# Build a wheel for distribution
maturin build --release
```

The project structure:

```
my_extension/
├── Cargo.toml          # Rust dependencies
├── pyproject.toml      # Python package metadata
├── src/
│   └── lib.rs          # Rust source code
└── python/
    └── my_extension/
        └── __init__.py # Python wrapper (optional)
```

### Releasing the GIL

Rust extensions can release the GIL for true parallelism:

```rust
use pyo3::prelude::*;

#[pyfunction]
fn heavy_computation(py: Python<'_>, data: Vec<f64>) -> f64 {
    // Release the GIL so other Python threads can run
    py.allow_threads(|| {
        // This runs without the GIL - true parallelism!
        data.iter()
            .map(|x| (x * x).sqrt().sin())
            .sum()
    })
}
```

### PyO3 Classes

```rust
use pyo3::prelude::*;

#[pyclass]
struct Point {
    #[pyo3(get, set)]
    x: f64,
    #[pyo3(get, set)]
    y: f64,
}

#[pymethods]
impl Point {
    #[new]
    fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }

    fn distance(&self, other: &Point) -> f64 {
        ((self.x - other.x).powi(2) + (self.y - other.y).powi(2)).sqrt()
    }

    fn __repr__(&self) -> String {
        format!("Point({}, {})", self.x, self.y)
    }
}
```

### Popular Rust-Python Libraries

| Library | Replaces | Speedup |
|---------|----------|--------|
| pydantic v2 | pydantic v1 | 5-50x |
| orjson | json | 3-10x |
| polars | pandas | 5-100x |
| ruff | flake8+isort | 10-100x |
| cryptography | PyCrypto | Safer + faster |
| tokenizers (HuggingFace) | Pure Python | 10-100x |

## Common Pitfalls

1. **Fighting the borrow checker** -- Rust's ownership system is unfamiliar to Python developers. Start with simple functions returning owned values before attempting complex lifetime management with references and borrowed data.
2. **Unnecessary data copying across the boundary** -- Converting a Python list to a Rust Vec copies all elements. For large datasets, use buffer protocol (PyO3's `PyBuffer`) or accept NumPy arrays directly to avoid costly copies.

## Best Practices

1. **Use maturin for builds** -- maturin handles cross-compilation, wheel building, and PyPI publishing with minimal configuration, making distribution painless.
2. **Release the GIL for CPU work** -- Use `py.allow_threads()` around CPU-intensive Rust code to enable true multi-threaded parallelism from Python.
3. **Keep the Python API Pythonic** -- Use `#[pyclass]` and `#[pymethods]` to expose idiomatic Python interfaces with `__repr__`, `__eq__`, and other dunder methods, hiding Rust complexity behind a familiar API.

## Summary

- PyO3 provides safe, ergonomic bindings between Rust and Python with automatic reference counting.
- maturin simplifies building and distributing Rust-based Python packages to a single command.
- Rust's compile-time memory and thread safety eliminates segfaults and data races in extensions.
- Use `py.allow_threads()` to release the GIL during CPU-intensive Rust code for true parallelism.
- Major libraries like pydantic, polars, and ruff demonstrate that Rust extensions can transform Python tool performance.

## Code Examples

**Demonstrating how Rust-based Python extensions (like orjson) provide dramatic speedups while maintaining a familiar Python API**

```python
# After building with maturin, use Rust extensions like regular Python
# maturin develop  (builds and installs in current venv)

# Example: using a hypothetical Rust extension
# from fast_math import sum_squares, Point

# Rust functions feel like native Python
# result = sum_squares(10_000_000)  # Runs at C speed

# Rust classes work like Python classes
# p1 = Point(0.0, 0.0)
# p2 = Point(3.0, 4.0)
# print(p1.distance(p2))  # 5.0
# print(p1)               # Point(0.0, 0.0)

# Practical example: orjson (real Rust-based library)
import json
import time

# orjson is a drop-in replacement written in Rust
try:
    import orjson
    data = {"users": [{"id": i, "name": f"user_{i}"} for i in range(10000)]}

    start = time.perf_counter()
    for _ in range(100):
        json.dumps(data)
    stdlib_time = time.perf_counter() - start

    start = time.perf_counter()
    for _ in range(100):
        orjson.dumps(data)
    orjson_time = time.perf_counter() - start

    print(f"json:   {stdlib_time*1000:.1f}ms")
    print(f"orjson: {orjson_time*1000:.1f}ms")
    print(f"Speedup: {stdlib_time/orjson_time:.1f}x")
except ImportError:
    print("Install orjson to see the Rust speedup: pip install orjson")
```


## Resources

- [Extending Python with C or C++](https://docs.python.org/3.14/extending/extending.html) — CPython's extension API that PyO3 builds upon
- [Building C and C++ Extensions](https://docs.python.org/3.14/extending/building.html) — Python 3.14 guide to building native extensions

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*