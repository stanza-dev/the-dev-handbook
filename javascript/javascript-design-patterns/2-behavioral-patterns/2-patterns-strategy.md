---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-strategy"
---

# The Strategy Pattern

## Introduction

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.

## Key Concepts

**Strategy**: An interchangeable algorithm encapsulated behind a common interface.

**Context**: The object that delegates work to a strategy.

**Runtime Selection**: Strategies can be swapped at runtime without changing client code.

## Real World Context

Payment processing (Stripe, PayPal, Apple Pay), form validation, sorting algorithms, and authentication methods all use Strategy. Any time a user picks from multiple approaches at runtime, you are likely using this pattern.

## Deep Dive

### Payment Processing

Payment strategies are stored as an object map of functions. The `PaymentProcessor` context delegates work to whichever strategy is currently set:

```javascript
const paymentStrategies = {
  creditCard: (amount) => {
    console.log(`Charging $${amount} to credit card`);
    return { success: true, method: 'credit' };
  },
  paypal: (amount) => {
    console.log(`Processing $${amount} via PayPal`);
    return { success: true, method: 'paypal' };
  }
};

class PaymentProcessor {
  #strategy;
  
  setStrategy(strategy) {
    this.#strategy = strategy;
  }
  
  process(amount) {
    if (!this.#strategy) throw new Error('No strategy set');
    return this.#strategy(amount);
  }
}

const processor = new PaymentProcessor();
processor.setStrategy(paymentStrategies.paypal);
processor.process(100);
```

The processor does not know which strategy it is using. Changing from `paypal` to `creditCard` requires only a different `setStrategy()` call, with no other code changes.

### Validation Strategies

Validation is another natural fit for strategies. Each validator is a pure function that returns a boolean, and the right one is selected by key:

```javascript
const validators = {
  email: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
  phone: (v) => /^\d{10}$/.test(v)
};

function validate(value, type) {
  return validators[type]?.(value) ?? false;
}
```

The optional chaining (`?.`) gracefully handles unknown types by returning `undefined`, which the nullish coalescing (`??`) converts to `false`.

## Common Pitfalls

1. **Over-engineering simple cases** — If you only have two strategies, an `if/else` is simpler than a full pattern.
2. **Strategy explosion** — Too many single-method strategies can clutter the codebase. Group related behaviors.
3. **Shared state between strategies** — Strategies should be stateless; passing state through context keeps them reusable.

## Best Practices

1. **Use plain objects for simple strategies** — A map of functions is often enough in JavaScript; full classes are rarely needed.
2. **Make strategies interchangeable** — All strategies must share the same interface (same parameters, same return shape).
3. **Document strategy options** — Consumers need to know which strategies exist and when to use each.

## Summary

Strategy pattern encapsulates interchangeable algorithms. Use for payment methods, validation, sorting. Prefer object maps for simple cases.

## Code Examples

**Strategy pattern for payment processing — strategies are swappable at runtime via setStrategy()**

```javascript
const paymentStrategies = {
  creditCard: (amount) => ({ success: true, method: 'credit', charged: amount }),
  paypal: (amount) => ({ success: true, method: 'paypal', charged: amount }),
  crypto: (amount) => ({ success: true, method: 'crypto', charged: amount })
};

class PaymentProcessor {
  #strategy;
  setStrategy(name) { this.#strategy = paymentStrategies[name]; }
  process(amount) {
    if (!this.#strategy) throw new Error('No strategy set');
    return this.#strategy(amount);
  }
}

const processor = new PaymentProcessor();
processor.setStrategy('paypal');
processor.process(99.99); // { success: true, method: 'paypal', charged: 99.99 }
```


## Resources

- [Refactoring Guru: Strategy](https://refactoring.guru/design-patterns/strategy) — Strategy pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*