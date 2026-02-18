---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-mocking"
---

# Mocking in Vitest

When your code depends on external services, use mocks.

## Mocking Functions

```javascript
import { vi, describe, test, expect } from 'vitest';
import { fetchUser } from './api';
import { UserService } from './userService';

// Mock the entire module
vi.mock('./api', () => ({
  fetchUser: vi.fn()
}));

describe('UserService', () => {
  test('loads user data', async () => {
    // Setup mock return value
    fetchUser.mockResolvedValue({ id: 1, name: 'Alice' });
    
    const service = new UserService();
    await service.loadUser(1);
    
    expect(fetchUser).toHaveBeenCalledWith(1);
    expect(service.user.name).toBe('Alice');
  });
  
  test('handles errors', async () => {
    fetchUser.mockRejectedValue(new Error('Network error'));
    
    const service = new UserService();
    await service.loadUser(1);
    
    expect(service.error).toBe('Network error');
  });
});
```

## Mocking fetch

```javascript
import { vi, beforeEach, afterEach } from 'vitest';

beforeEach(() => {
  global.fetch = vi.fn();
});

afterEach(() => {
  vi.restoreAllMocks();
});

test('fetches data', async () => {
  fetch.mockResolvedValue({
    ok: true,
    json: async () => ({ data: 'test' })
  });
  
  const result = await myFetchFunction();
  
  expect(fetch).toHaveBeenCalledWith('/api/data');
  expect(result.data).toBe('test');
});
```

## Spying on Methods

```javascript
test('calls callback on success', () => {
  const callback = vi.fn();
  const service = new Service(callback);
  
  service.doSomething();
  
  expect(callback).toHaveBeenCalled();
  expect(callback).toHaveBeenCalledWith({ success: true });
});
```

## Timer Mocks

```javascript
import { vi, beforeEach, afterEach } from 'vitest';

beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

test('debounces calls', async () => {
  const fn = vi.fn();
  const debounced = debounce(fn, 100);
  
  debounced();
  debounced();
  debounced();
  
  expect(fn).not.toHaveBeenCalled();
  
  vi.advanceTimersByTime(100);
  
  expect(fn).toHaveBeenCalledTimes(1);
});
```

📖 [Vitest mocking](https://vitest.dev/guide/mocking)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*