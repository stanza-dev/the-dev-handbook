---
source_course: "vue-testing"
source_lesson: "vue-testing-user-interactions"
---

# Testing User Interactions

Test how components respond to user actions like clicks, input, and form submissions.

## Triggering Events

```typescript
import { mount } from '@vue/test-utils'

describe('Button', () => {
  it('handles click', async () => {
    const wrapper = mount(Counter)
    
    // Must await - triggers DOM update
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.text()).toContain('Count: 1')
  })
})
```

## Common Events

```typescript
// Click
await wrapper.find('button').trigger('click')

// Double click
await wrapper.find('div').trigger('dblclick')

// Keyboard
await wrapper.find('input').trigger('keydown.enter')
await wrapper.find('input').trigger('keyup.escape')

// Focus/Blur
await wrapper.find('input').trigger('focus')
await wrapper.find('input').trigger('blur')

// Mouse
await wrapper.find('div').trigger('mouseenter')
await wrapper.find('div').trigger('mouseleave')

// Form
await wrapper.find('form').trigger('submit')
```

## Event Modifiers

```typescript
// With modifiers
await wrapper.find('form').trigger('submit.prevent')
await wrapper.find('input').trigger('keydown.enter.ctrl')
```

## Setting Input Values

```typescript
import { mount } from '@vue/test-utils'
import Form from './Form.vue'

describe('Form', () => {
  it('updates v-model', async () => {
    const wrapper = mount(Form)
    const input = wrapper.find('input')
    
    // Set value
    await input.setValue('test@example.com')
    
    expect(wrapper.vm.email).toBe('test@example.com')
  })
  
  it('handles checkbox', async () => {
    const wrapper = mount(Form)
    const checkbox = wrapper.find('input[type="checkbox"]')
    
    await checkbox.setValue(true)  // Check
    expect(wrapper.vm.agreed).toBe(true)
    
    await checkbox.setValue(false)  // Uncheck
    expect(wrapper.vm.agreed).toBe(false)
  })
  
  it('handles select', async () => {
    const wrapper = mount(Form)
    const select = wrapper.find('select')
    
    await select.setValue('option2')
    
    expect(wrapper.vm.selected).toBe('option2')
  })
})
```

## Testing Emitted Events

```typescript
import { mount } from '@vue/test-utils'
import SubmitButton from './SubmitButton.vue'

describe('SubmitButton', () => {
  it('emits submit event on click', async () => {
    const wrapper = mount(SubmitButton)
    
    await wrapper.find('button').trigger('click')
    
    // Check if event was emitted
    expect(wrapper.emitted()).toHaveProperty('submit')
    
    // Check event payload
    expect(wrapper.emitted('submit')).toHaveLength(1)
    expect(wrapper.emitted('submit')[0]).toEqual([{ id: 1 }])
  })
  
  it('emits multiple events', async () => {
    const wrapper = mount(Counter)
    
    await wrapper.find('.increment').trigger('click')
    await wrapper.find('.increment').trigger('click')
    
    const changeEvents = wrapper.emitted('change')
    expect(changeEvents).toHaveLength(2)
    expect(changeEvents[0]).toEqual([1])
    expect(changeEvents[1]).toEqual([2])
  })
})
```

## Testing Form Submission

```typescript
import { mount } from '@vue/test-utils'
import LoginForm from './LoginForm.vue'

describe('LoginForm', () => {
  it('submits form data', async () => {
    const wrapper = mount(LoginForm)
    
    await wrapper.find('#email').setValue('test@example.com')
    await wrapper.find('#password').setValue('password123')
    await wrapper.find('form').trigger('submit.prevent')
    
    expect(wrapper.emitted('submit')).toBeTruthy()
    expect(wrapper.emitted('submit')[0][0]).toEqual({
      email: 'test@example.com',
      password: 'password123'
    })
  })
  
  it('shows validation errors', async () => {
    const wrapper = mount(LoginForm)
    
    // Submit empty form
    await wrapper.find('form').trigger('submit.prevent')
    
    expect(wrapper.find('.error').exists()).toBe(true)
    expect(wrapper.find('.error').text()).toContain('Email is required')
  })
})
```

## Waiting for Async Updates

```typescript
import { mount, flushPromises } from '@vue/test-utils'

describe('AsyncComponent', () => {
  it('loads data', async () => {
    const wrapper = mount(UserList)
    
    // Wait for all promises to resolve
    await flushPromises()
    
    expect(wrapper.findAll('li')).toHaveLength(3)
  })
})
```

## Resources

- [Testing Events](https://test-utils.vuejs.org/guide/essentials/event-handling.html) — Testing event handling

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*