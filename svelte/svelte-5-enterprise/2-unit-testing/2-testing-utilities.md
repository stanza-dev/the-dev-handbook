---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-testing-utilities"
---

# Testing Pure Functions

Pure utility functions are the easiest to test - no setup, no mocking.

## Formatting Functions

```javascript
// utils/format.js
export function formatCurrency(amount, currency = 'USD') {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount);
}

export function formatDate(date) {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  }).format(new Date(date));
}
```

```javascript
// utils/format.test.js
import { formatCurrency, formatDate } from './format';

describe('formatCurrency', () => {
  test('formats USD by default', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });
  
  test('handles zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });
  
  test('supports other currencies', () => {
    expect(formatCurrency(100, 'EUR')).toBe('€100.00');
  });
});
```

## Validation Functions

```javascript
// utils/validation.js
export function validateEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

export function validatePassword(password) {
  const errors = [];
  if (password.length < 8) errors.push('Must be at least 8 characters');
  if (!/[A-Z]/.test(password)) errors.push('Must contain uppercase');
  if (!/[0-9]/.test(password)) errors.push('Must contain number');
  return { valid: errors.length === 0, errors };
}
```

```javascript
// utils/validation.test.js
describe('validateEmail', () => {
  test.each([
    ['user@example.com', true],
    ['invalid', false],
    ['missing@domain', false],
    ['@nodomain.com', false],
  ])('validateEmail(%s) returns %s', (email, expected) => {
    expect(validateEmail(email)).toBe(expected);
  });
});

describe('validatePassword', () => {
  test('rejects short passwords', () => {
    const result = validatePassword('Short1');
    expect(result.valid).toBe(false);
    expect(result.errors).toContain('Must be at least 8 characters');
  });
  
  test('accepts valid password', () => {
    const result = validatePassword('ValidPass123');
    expect(result.valid).toBe(true);
  });
});
```

📖 [Vitest test.each](https://vitest.dev/api/#test-each)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*