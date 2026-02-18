---
source_course: "vue-testing"
source_lesson: "vue-testing-testing-patterns"
---

# Testing Patterns and Anti-Patterns

Write tests that are maintainable, readable, and actually catch bugs.

## The AAA Pattern

Structure tests with Arrange, Act, Assert:

```typescript
describe('Counter', () => {
  it('increments count when button clicked', async () => {
    // Arrange - Set up the test
    const wrapper = mount(Counter, {
      props: { initialCount: 5 }
    })
    
    // Act - Perform the action
    await wrapper.find('button').trigger('click')
    
    // Assert - Verify the result
    expect(wrapper.find('.count').text()).toBe('6')
  })
})
```

## Test Behavior, Not Implementation

```typescript
// ❌ Testing implementation details
it('calls increment method', async () => {
  const wrapper = mount(Counter)
  const spy = vi.spyOn(wrapper.vm, 'increment')
  
  await wrapper.find('button').trigger('click')
  
  expect(spy).toHaveBeenCalled()  // Brittle!
})

// ✅ Testing behavior (what the user sees)
it('displays incremented count after click', async () => {
  const wrapper = mount(Counter, { props: { initialCount: 0 } })
  
  await wrapper.find('button').trigger('click')
  
  expect(wrapper.text()).toContain('1')  // User-visible behavior
})
```

## Test Isolation

```typescript
describe('UserProfile', () => {
  let wrapper: VueWrapper
  
  // Fresh setup for each test
  beforeEach(() => {
    wrapper = mount(UserProfile, {
      props: { userId: 1 }
    })
  })
  
  afterEach(() => {
    wrapper.unmount()
  })
  
  it('displays user name', () => {
    // Each test starts fresh
  })
})
```

## Descriptive Test Names

```typescript
// ❌ Vague names
it('works', () => {})
it('handles error', () => {})

// ✅ Descriptive names
it('displays error message when API returns 404', () => {})
it('disables submit button while form is submitting', () => {})
it('redirects to dashboard after successful login', () => {})
```

## Testing Edge Cases

```typescript
describe('SearchInput', () => {
  it('shows no results message for empty search', async () => {
    const wrapper = mount(SearchInput)
    await wrapper.find('input').setValue('')
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.text()).toContain('Enter a search term')
  })
  
  it('handles special characters in search', async () => {
    const wrapper = mount(SearchInput)
    await wrapper.find('input').setValue('<script>alert(1)</script>')
    
    // Should sanitize/escape
    expect(wrapper.find('.search-term').text()).not.toContain('<script>')
  })
  
  it('trims whitespace from search query', async () => {
    const wrapper = mount(SearchInput)
    await wrapper.find('input').setValue('  hello world  ')
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.emitted('search')?.[0]).toEqual(['hello world'])
  })
})
```

## Anti-Patterns to Avoid

### 1. Testing Props Directly

```typescript
// ❌ Don't test that Vue works
it('receives props', () => {
  const wrapper = mount(Button, { props: { label: 'Click' } })
  expect(wrapper.props('label')).toBe('Click')  // Pointless!
})

// ✅ Test how props affect the component
it('displays the label text', () => {
  const wrapper = mount(Button, { props: { label: 'Click' } })
  expect(wrapper.text()).toBe('Click')
})
```

### 2. Over-Mocking

```typescript
// ❌ Mocking everything
it('renders', () => {
  const wrapper = mount(Component, {
    global: {
      stubs: ['ChildA', 'ChildB', 'ChildC', 'ChildD']
    }
  })
  // Testing a shell with no real behavior
})

// ✅ Only mock what's necessary
it('displays user data', () => {
  const wrapper = mount(UserCard, {
    props: { user: mockUser },
    global: {
      stubs: ['ExpensiveChart']  // Only stub expensive/external
    }
  })
})
```

## Resources

- [Vue Test Utils Guide](https://test-utils.vuejs.org/guide/) — Official Vue Test Utils documentation

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*