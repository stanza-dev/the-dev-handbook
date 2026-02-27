---
source_course: "python-architecture"
source_lesson: "python-architecture-strategy-template"
---

# Strategy & Template Method Patterns

## Introduction

The **Strategy** and **Template Method** patterns both address the problem of varying behavior, but they do it in opposite ways. Strategy uses *composition* (swap an algorithm object), while Template Method uses *inheritance* (override specific steps). Python's Protocols and first-class functions make both patterns lightweight.

## Key Concepts

- **Strategy** — define a family of interchangeable algorithms behind a common interface (Protocol). The client picks the algorithm at runtime.
- **Template Method** — define the skeleton of an algorithm in a base class, deferring certain steps to subclasses via abstract methods.

## Real World Context

| Pattern | Where You See It |
|---------|------------------|
| Strategy | Payment processing (Stripe vs PayPal), compression (gzip vs brotli), sorting key functions |
| Template Method | Data pipeline ETL (extract/transform/load), test frameworks (setUp/tearDown), report generators |

## Deep Dive

### Strategy with Protocols

```python
from typing import Protocol

class CompressionStrategy(Protocol):
    def compress(self, data: bytes) -> bytes: ...
    def decompress(self, data: bytes) -> bytes: ...

class GzipCompression:
    def compress(self, data: bytes) -> bytes:
        import gzip
        return gzip.compress(data)

    def decompress(self, data: bytes) -> bytes:
        import gzip
        return gzip.decompress(data)

class NoCompression:
    def compress(self, data: bytes) -> bytes:
        return data

    def decompress(self, data: bytes) -> bytes:
        return data

class FileStore:
    def __init__(self, strategy: CompressionStrategy) -> None:
        self._strategy = strategy

    def save(self, path: str, data: bytes) -> None:
        compressed = self._strategy.compress(data)
        with open(path, "wb") as f:
            f.write(compressed)

# Swap algorithms at runtime
store = FileStore(GzipCompression())
store.save("data.gz", b"hello world")
```

### Strategy with Callable (Pythonic shortcut)

When the strategy is a single function, you can skip the Protocol entirely:

```python
from typing import Callable

type SortKey[T] = Callable[[T], object]

def sorted_users(users: list[dict], key: SortKey[dict]) -> list[dict]:
    return sorted(users, key=key)

# Strategy is just a function
by_name = lambda u: u["name"]
by_age = lambda u: u["age"]

result = sorted_users(users, key=by_name)
```

### Template Method with ABC

```python
from abc import ABC, abstractmethod

class DataPipeline(ABC):
    """Template: the run() skeleton is fixed; steps vary."""

    def run(self, source: str) -> None:
        raw = self.extract(source)
        clean = self.transform(raw)
        self.load(clean)

    @abstractmethod
    def extract(self, source: str) -> list[dict]: ...

    @abstractmethod
    def transform(self, data: list[dict]) -> list[dict]: ...

    @abstractmethod
    def load(self, data: list[dict]) -> None: ...

class CSVPipeline(DataPipeline):
    def extract(self, source: str) -> list[dict]:
        import csv, io
        reader = csv.DictReader(io.StringIO(source))
        return list(reader)

    def transform(self, data: list[dict]) -> list[dict]:
        return [{k: v.strip() for k, v in row.items()} for row in data]

    def load(self, data: list[dict]) -> None:
        for row in data:
            print(row)
```

## Common Pitfalls

1. **Over-engineering Strategy** — if you will only ever have two strategies, a simple `if/else` is fine. Introduce the pattern when a third variant appears.
2. **Template fragility** — deep inheritance hierarchies make Template Method hard to follow. Keep the inheritance to one level.
3. **Mixing the two** — Strategy is composition-based; Template Method is inheritance-based. Combining them in the same class creates confusion.

## Best Practices

- Use **Protocol** for Strategy when the algorithm has multiple methods.
- Use a plain **Callable** for Strategy when the algorithm is a single function.
- Use **ABC + abstractmethod** for Template Method to enforce subclass contracts.
- Prefer Strategy over Template Method when you need to **switch behavior at runtime**.

## Summary

Strategy externalizes varying behavior into swappable objects, leveraging Python Protocols for type safety without inheritance. Template Method locks the algorithm skeleton in a base class while letting subclasses customize individual steps. Choose Strategy for runtime flexibility and Template Method for enforcing a consistent workflow.

## Code Examples

**Strategy pattern with Protocol for pricing**

```python
from typing import Protocol

class PricingStrategy(Protocol):
    def calculate(self, base_price: float) -> float: ...

class RegularPricing:
    def calculate(self, base_price: float) -> float:
        return base_price

class PremiumDiscount:
    def __init__(self, discount: float = 0.2) -> None:
        self._discount = discount

    def calculate(self, base_price: float) -> float:
        return base_price * (1 - self._discount)

class SeasonalSale:
    def calculate(self, base_price: float) -> float:
        return base_price * 0.7

class Order:
    def __init__(self, pricing: PricingStrategy) -> None:
        self._pricing = pricing

    def total(self, base_price: float) -> float:
        return self._pricing.calculate(base_price)

# Swap pricing at runtime
order = Order(PremiumDiscount(discount=0.15))
print(order.total(100.0))  # 85.0

order = Order(SeasonalSale())
print(order.total(100.0))  # 70.0
```

**Template Method for report generation**

```python
from abc import ABC, abstractmethod

class ReportGenerator(ABC):
    """Template Method: generate() defines the skeleton."""

    def generate(self, data: list[dict]) -> str:
        header = self.build_header()
        body = self.build_body(data)
        footer = self.build_footer()
        return f"{header}\n{body}\n{footer}"

    @abstractmethod
    def build_header(self) -> str: ...

    @abstractmethod
    def build_body(self, data: list[dict]) -> str: ...

    def build_footer(self) -> str:
        return "--- End of Report ---"

class MarkdownReport(ReportGenerator):
    def build_header(self) -> str:
        return "# Sales Report"

    def build_body(self, data: list[dict]) -> str:
        lines = [f"- {row['item']}: ${row['amount']}" for row in data]
        return "\n".join(lines)

report = MarkdownReport()
print(report.generate([{"item": "Widget", "amount": 29.99}]))
```


## Resources

- [Python abc module](https://docs.python.org/3/library/abc.html) — Abstract Base Classes for Template Method implementations
- [PEP 544 — Protocols](https://peps.python.org/pep-0544/) — Structural subtyping for Strategy pattern interfaces

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*