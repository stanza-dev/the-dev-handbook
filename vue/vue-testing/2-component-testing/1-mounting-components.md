---
source_course: "vue-testing"
source_lesson: "vue-testing-mounting-components"
---

# Mounting Components

Vue Test Utils provides `mount` and `shallowMount` to render components for testing.

## Basic Mounting

```typescript
import { mount } from '@vue/test-utils'
import MyComponent from './MyComponent.vue'

describe('MyComponent', () => {
  it('renders correctly', () => {
    const wrapper = mount(MyComponent)
    
    expect(wrapper.exists()).toBe(true)
    expect(wrapper.text()).toContain('Hello')
  })
})
```

## mount vs shallowMount

```typescript
import { mount, shallowMount } from '@vue/test-utils'

// mount: renders child components
const wrapper = mount(Parent)
// All children are fully rendered

// shallowMount: stubs child components
const shallowWrapper = shallowMount(Parent)
// Children are stubbed (not rendered)
```

Use `mount` for integration tests, `shallowMount` for isolated unit tests.

## Mounting Options

```typescript
const wrapper = mount(MyComponent, {
  // Props
  props: {
    title: 'Hello',
    count: 5
  },
  
  // Slots
  slots: {
    default: 'Default content',
    header: '<h1>Header</h1>',
    footer: FooterComponent
  },
  
  // Global options
  global: {
    plugins: [router, pinia],
    components: { MyGlobalComponent },
    directives: { focus: focusDirective },
    mocks: {
      $route: { params: { id: 1 } }
    },
    stubs: {
      RouterLink: true,
      MyExpensiveComponent: true
    },
    provide: {
      theme: 'dark'
    }
  },
  
  // Attributes
  attrs: {
    class: 'test-class',
    id: 'test-id'
  }
})
```

## Finding Elements

```typescript
const wrapper = mount(MyComponent)

// By CSS selector
const button = wrapper.find('button')
const form = wrapper.find('.form-class')
const input = wrapper.find('#email-input')

// Multiple elements
const items = wrapper.findAll('li')
expect(items).toHaveLength(3)

// By component
import ChildComponent from './ChildComponent.vue'
const child = wrapper.findComponent(ChildComponent)
const children = wrapper.findAllComponents(ChildComponent)

// By data-testid (recommended)
const submitBtn = wrapper.find('[data-testid="submit-button"]')
```

## Wrapper Properties

```typescript
const wrapper = mount(MyComponent)

// Text content
wrapper.text()  // All text content

// HTML
wrapper.html()  // Rendered HTML

// Element
wrapper.element  // DOM element

// VM (component instance)
wrapper.vm  // Access component internals

// Props
wrapper.props()  // All props
wrapper.props('title')  // Specific prop

// Attributes
wrapper.attributes()  // All attributes
wrapper.attributes('id')  // Specific attribute

// Classes
wrapper.classes()  // Array of classes
wrapper.classes('active')  // Boolean check

// Existence
wrapper.exists()  // Boolean
```

## Testing Props

```typescript
import { mount } from '@vue/test-utils'
import Greeting from './Greeting.vue'

describe('Greeting', () => {
  it('renders name prop', () => {
    const wrapper = mount(Greeting, {
      props: { name: 'Alice' }
    })
    
    expect(wrapper.text()).toContain('Hello, Alice')
  })
  
  it('uses default prop', () => {
    const wrapper = mount(Greeting)
    
    expect(wrapper.text()).toContain('Hello, Guest')
  })
  
  it('updates when prop changes', async () => {
    const wrapper = mount(Greeting, {
      props: { name: 'Alice' }
    })
    
    await wrapper.setProps({ name: 'Bob' })
    
    expect(wrapper.text()).toContain('Hello, Bob')
  })
})
```

## Testing Slots

```typescript
import { mount } from '@vue/test-utils'
import Card from './Card.vue'

describe('Card', () => {
  it('renders default slot', () => {
    const wrapper = mount(Card, {
      slots: {
        default: '<p>Card content</p>'
      }
    })
    
    expect(wrapper.html()).toContain('<p>Card content</p>')
  })
  
  it('renders named slots', () => {
    const wrapper = mount(Card, {
      slots: {
        header: '<h2>Title</h2>',
        footer: '<button>Save</button>'
      }
    })
    
    expect(wrapper.find('h2').text()).toBe('Title')
    expect(wrapper.find('button').text()).toBe('Save')
  })
})
```

## Resources

- [Vue Test Utils](https://test-utils.vuejs.org/) — Official Vue Test Utils documentation

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*