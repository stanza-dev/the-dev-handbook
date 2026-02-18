---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-testing-runes"
---

# Unit Testing Svelte 5 Logic

Svelte 5's universal reactivity makes unit testing a joy. Your business logic in `.svelte.js` files can be tested like any JavaScript.

## Testing a State Class

```javascript
// counter.svelte.js
export class Counter {
  count = $state(0);
  
  increment() {
    this.count++;
  }
  
  decrement() {
    this.count--;
  }
  
  reset() {
    this.count = 0;
  }
}
```

```javascript
// counter.test.js
import { describe, test, expect, beforeEach } from 'vitest';
import { Counter } from './counter.svelte.js';

describe('Counter', () => {
  let counter;
  
  beforeEach(() => {
    counter = new Counter();
  });
  
  test('starts at zero', () => {
    expect(counter.count).toBe(0);
  });
  
  test('increments', () => {
    counter.increment();
    expect(counter.count).toBe(1);
  });
  
  test('decrements', () => {
    counter.count = 5;
    counter.decrement();
    expect(counter.count).toBe(4);
  });
  
  test('resets to zero', () => {
    counter.count = 100;
    counter.reset();
    expect(counter.count).toBe(0);
  });
});
```

## Testing Derived Values

```javascript
// cart.svelte.js
export class Cart {
  items = $state([]);
  
  get itemCount() {
    return this.items.length;
  }
  
  get total() {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
  
  add(product, quantity = 1) {
    this.items = [...this.items, { ...product, quantity }];
  }
}
```

```javascript
// cart.test.js
describe('Cart', () => {
  test('calculates total correctly', () => {
    const cart = new Cart();
    cart.add({ id: 1, price: 10 }, 2);  // $20
    cart.add({ id: 2, price: 5 }, 3);   // $15
    
    expect(cart.total).toBe(35);
    expect(cart.itemCount).toBe(2);
  });
});
```

## Note on Reactivity in Tests

Vitest may need special setup for Svelte's runtime. For simple class tests, it often "just works". For complex reactive scenarios, use Vitest's browser mode or component tests.

📖 [Vitest documentation](https://vitest.dev/)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*