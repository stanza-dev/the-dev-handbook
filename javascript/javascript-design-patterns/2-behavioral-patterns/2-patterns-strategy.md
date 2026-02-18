---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-strategy"
---

# The Strategy Pattern

## Introduction

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.

## Deep Dive

### Payment Processing

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

### Validation Strategies

```javascript
const validators = {
  email: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
  phone: (v) => /^\d{10}$/.test(v)
};

function validate(value, type) {
  return validators[type]?.(value) ?? false;
}
```

## Summary

Strategy pattern encapsulates interchangeable algorithms. Use for payment methods, validation, sorting. Prefer object maps for simple cases.

## Resources

- [Refactoring Guru: Strategy](https://refactoring.guru/design-patterns/strategy) — Strategy pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*