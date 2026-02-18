---
source_course: "python-performance"
source_lesson: "python-performance-pyo3"
---

# PyO3: Python + Rust

Rust provides memory safety without garbage collection.

## Why Rust?

- Memory safety guarantees
- No null pointer exceptions
- Thread safety
- Performance matching C/C++

## Popular Rust Python Libraries

- **pydantic-core**: Data validation
- **cryptography**: Cryptographic operations
- **orjson**: Fast JSON
- **polars**: DataFrames (like Pandas)
- **ruff**: Python linter

## Basic PyO3 Example

```rust
// src/lib.rs
use pyo3::prelude::*;

#[pyfunction]
fn sum_as_string(a: i64, b: i64) -> String {
    (a + b).to_string()
}

#[pymodule]
fn my_module(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}
```

## Building with maturin

```bash
# Install maturin
pip install maturin

# Create new project
maturin new my_project

# Build and install
cd my_project
maturin develop
```

## Using from Python

```python
import my_module

result = my_module.sum_as_string(10, 20)
print(result)  # "30"
```

## Code Examples

**orjson benchmark**

```python
# Example: Using orjson (Rust-based JSON)
import json
import orjson  # pip install orjson
import timeit

data = {"users": [{"name": f"user{i}", "id": i} for i in range(1000)]}

# Benchmark
std_time = timeit.timeit(lambda: json.dumps(data), number=1000)
orjson_time = timeit.timeit(lambda: orjson.dumps(data), number=1000)

print(f"json:   {std_time:.4f}s")
print(f"orjson: {orjson_time:.4f}s")
print(f"Speedup: {std_time/orjson_time:.1f}x")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*