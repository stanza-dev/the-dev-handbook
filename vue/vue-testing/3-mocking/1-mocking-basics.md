---
source_course: "vue-testing"
source_lesson: "vue-testing-mocking-basics"
---

# Mocking with Vitest

Mocking isolates your tests from external dependencies like APIs, modules, and timers.

## vi.fn() - Mock Functions

```typescript
import { describe, it, expect, vi } from 'vitest'

describe('Mock Functions', () => {
  it('tracks calls', () => {
    const mockFn = vi.fn()
    
    mockFn('arg1', 'arg2')
    mockFn('arg3')
    
    expect(mockFn).toHaveBeenCalled()
    expect(mockFn).toHaveBeenCalledTimes(2)
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2')
    expect(mockFn).toHaveBeenLastCalledWith('arg3')
  })
  
  it('returns values', () => {
    const mockFn = vi.fn()
      .mockReturnValue('default')
      .mockReturnValueOnce('first')
      .mockReturnValueOnce('second')
    
    expect(mockFn()).toBe('first')
    expect(mockFn()).toBe('second')
    expect(mockFn()).toBe('default')
  })
  
  it('implements custom behavior', () => {
    const mockFn = vi.fn((x: number) => x * 2)
    
    expect(mockFn(5)).toBe(10)
  })
})
```

## vi.spyOn() - Spy on Methods

```typescript
import { vi } from 'vitest'
import * as utils from './utils'

describe('Spying', () => {
  it('spies on method', () => {
    const spy = vi.spyOn(utils, 'formatDate')
    
    utils.formatDate(new Date())
    
    expect(spy).toHaveBeenCalled()
    
    spy.mockRestore()  // Restore original
  })
  
  it('mocks implementation', () => {
    const spy = vi.spyOn(console, 'log').mockImplementation(() => {})
    
    console.log('test')  // Doesn't actually log
    
    expect(spy).toHaveBeenCalledWith('test')
    
    spy.mockRestore()
  })
})
```

## vi.mock() - Mock Modules

```typescript
import { vi } from 'vitest'

// Mock entire module
vi.mock('./api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: 1, name: 'Test' }),
  fetchPosts: vi.fn().mockResolvedValue([])
}))

import { fetchUser } from './api'  // Now mocked

describe('With mocked API', () => {
  it('uses mock', async () => {
    const user = await fetchUser(1)
    expect(user.name).toBe('Test')
  })
})
```

## Mocking fetch

```typescript
import { vi, beforeEach } from 'vitest'

describe('API calls', () => {
  beforeEach(() => {
    // Mock global fetch
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ data: 'test' })
    })
  })
  
  it('fetches data', async () => {
    const response = await fetch('/api/data')
    const data = await response.json()
    
    expect(data).toEqual({ data: 'test' })
    expect(fetch).toHaveBeenCalledWith('/api/data')
  })
})
```

## Mocking Timers

```typescript
import { vi, beforeEach, afterEach } from 'vitest'

describe('Timers', () => {
  beforeEach(() => {
    vi.useFakeTimers()
  })
  
  afterEach(() => {
    vi.useRealTimers()
  })
  
  it('handles setTimeout', () => {
    const callback = vi.fn()
    
    setTimeout(callback, 1000)
    
    expect(callback).not.toHaveBeenCalled()
    
    vi.advanceTimersByTime(1000)
    
    expect(callback).toHaveBeenCalled()
  })
  
  it('handles setInterval', () => {
    const callback = vi.fn()
    
    setInterval(callback, 100)
    
    vi.advanceTimersByTime(350)
    
    expect(callback).toHaveBeenCalledTimes(3)
  })
  
  it('runs all timers', () => {
    const callback = vi.fn()
    
    setTimeout(callback, 5000)
    
    vi.runAllTimers()
    
    expect(callback).toHaveBeenCalled()
  })
})
```

## Mocking in Vue Components

```typescript
import { mount } from '@vue/test-utils'
import { vi } from 'vitest'
import UserProfile from './UserProfile.vue'
import * as api from '@/api'

vi.mock('@/api')

describe('UserProfile', () => {
  it('loads user data', async () => {
    vi.mocked(api.fetchUser).mockResolvedValue({
      id: 1,
      name: 'Alice'
    })
    
    const wrapper = mount(UserProfile, {
      props: { userId: 1 }
    })
    
    await flushPromises()
    
    expect(api.fetchUser).toHaveBeenCalledWith(1)
    expect(wrapper.text()).toContain('Alice')
  })
})
```

## Clearing Mocks

```typescript
afterEach(() => {
  vi.clearAllMocks()     // Clear call history
  vi.resetAllMocks()     // Clear + reset implementations
  vi.restoreAllMocks()   // Restore original implementations
})
```

## Resources

- [Vitest Mocking](https://vitest.dev/guide/mocking.html) — Vitest mocking guide

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*