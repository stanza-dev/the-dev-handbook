---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-strategy-pattern"
---

# Strategy Pattern with Interfaces

## Introduction
The Strategy pattern defines a family of interchangeable algorithms behind a common interface. TypeScript interfaces make this pattern type-safe: the compiler ensures every strategy implements the required methods, and consumers work with the interface without knowing which concrete strategy is in use.

## Key Concepts
- **Strategy Interface**: A contract that all algorithm variants must implement.
- **Context**: The class that holds a reference to a strategy and delegates work to it.
- **Runtime Swapping**: Strategies can be switched at runtime without changing the context's code.

## Real World Context
Payment processing (Stripe, PayPal, bank transfer), sorting algorithms, compression formats, and authentication methods (OAuth, JWT, API key) are all real-world strategy pattern applications. Each variant implements the same interface but with different internals.

## Deep Dive
### Defining the Strategy Interface

```typescript
interface CompressionStrategy {
    compress(data: Buffer): Buffer;
    decompress(data: Buffer): Buffer;
    readonly name: string;
}
```

### Implementing Strategies

```typescript
class GzipStrategy implements CompressionStrategy {
    readonly name = "gzip";
    compress(data: Buffer): Buffer { /* gzip logic */ return data; }
    decompress(data: Buffer): Buffer { /* gunzip logic */ return data; }
}

class BrotliStrategy implements CompressionStrategy {
    readonly name = "brotli";
    compress(data: Buffer): Buffer { /* brotli logic */ return data; }
    decompress(data: Buffer): Buffer { /* brotli decompress */ return data; }
}
```

### Context Class

```typescript
class FileCompressor {
    constructor(private strategy: CompressionStrategy) {}

    setStrategy(strategy: CompressionStrategy): void {
        this.strategy = strategy;
    }

    compressFile(data: Buffer): Buffer {
        console.log(`Compressing with ${this.strategy.name}`);
        return this.strategy.compress(data);
    }
}

const compressor = new FileCompressor(new GzipStrategy());
compressor.compressFile(data); // Uses gzip
compressor.setStrategy(new BrotliStrategy());
compressor.compressFile(data); // Now uses brotli
```

### Function-Based Strategies

In TypeScript, you can also use function types instead of classes:

```typescript
type SortStrategy<T> = (items: T[]) => T[];

const alphabetical: SortStrategy<string> = (items) => [...items].sort();
const byLength: SortStrategy<string> = (items) => [...items].sort((a, b) => a.length - b.length);

function displaySorted(items: string[], strategy: SortStrategy<string>): void {
    const sorted = strategy(items);
    console.log(sorted);
}

displaySorted(["banana", "fig", "apple"], byLength); // ["fig", "apple", "banana"]
```

## Common Pitfalls
1. **Leaking strategy internals** — The context should only interact with the strategy through the interface. Accessing strategy-specific properties breaks the abstraction.
2. **Over-engineering** — If you have only two variants and they are unlikely to change, a simple if/else may be clearer than the full Strategy pattern.

## Best Practices
1. **Prefer function strategies for simple cases** — When the strategy is a single function, use a type alias instead of a full interface and class hierarchy.
2. **Use dependency injection** — Pass strategies via constructor parameters for testability and flexibility.

## Summary
- The Strategy pattern encapsulates interchangeable algorithms behind a common interface.
- TypeScript interfaces guarantee every strategy implements the required contract.
- Function-based strategies are a lightweight alternative to class-based ones.
- Use dependency injection to swap strategies at construction time.

## Code Examples

**Function-based strategies for pricing — swap the pricing algorithm without changing the calculation logic**

```typescript
// Function-based strategy pattern
type PricingStrategy = (basePrice: number) => number;

const regularPricing: PricingStrategy = (price) => price;
const premiumDiscount: PricingStrategy = (price) => price * 0.8;
const bulkDiscount: PricingStrategy = (price) => price * 0.6;

function calculateTotal(items: number[], strategy: PricingStrategy): number {
    return items.reduce((sum, price) => sum + strategy(price), 0);
}

const prices = [100, 200, 50];
calculateTotal(prices, regularPricing);   // 350
calculateTotal(prices, premiumDiscount);  // 280
calculateTotal(prices, bulkDiscount);     // 210
```


## Resources

- [Refactoring Guru: Strategy Pattern](https://refactoring.guru/design-patterns/strategy) — Visual explanation of the Strategy pattern with examples in multiple languages

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*