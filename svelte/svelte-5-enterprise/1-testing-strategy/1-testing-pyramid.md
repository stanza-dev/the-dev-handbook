---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-testing-pyramid"
---

# Understanding the Testing Pyramid

Not all tests are created equal. The testing pyramid helps you invest testing effort wisely.

```
                    /\
                   /  \
                  / E2E \
                 / Tests  \       ← Few, slow, expensive
                /──────────\
               /             \
              /  Integration  \   ← Some, medium speed
             /     Tests       \
            /───────────────────\
           /                     \
          /      Unit Tests       \ ← Many, fast, cheap
         /─────────────────────────\
```

## Test Types in Svelte 5

### Unit Tests
- Test pure functions, utilities
- Test Runes classes (`.svelte.js` files)
- Test stores and derived state logic
- **Fast, isolated, deterministic**

```javascript
// math.svelte.js
export class Calculator {
  result = $state(0);
  
  add(n) { this.result += n; }
  multiply(n) { this.result *= n; }
}

// math.test.js
test('calculator adds correctly', () => {
  const calc = new Calculator();
  calc.add(5);
  expect(calc.result).toBe(5);
});
```

### Component/Integration Tests
- Render components in isolation
- Test component behavior and user interactions
- Use Svelte Testing Library
- **Medium speed, test user-facing behavior**

### E2E Tests
- Run full app in real browser
- Test complete user flows
- Use Playwright
- **Slow, but high confidence**

## Svelte 5's Testing Advantage

Because Runes work in `.svelte.js` files, you can test business logic as unit tests without rendering components:

```javascript
// cart.svelte.js - Your business logic
export class Cart {
  items = $state([]);
  
  add(product) { this.items = [...this.items, product]; }
  
  get total() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// cart.test.js - Unit test, no DOM needed!
test('calculates total', () => {
  const cart = new Cart();
  cart.add({ price: 10 });
  cart.add({ price: 20 });
  expect(cart.total).toBe(30);
});
```

📖 [Testing best practices](https://testing-library.com/docs/guiding-principles)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*