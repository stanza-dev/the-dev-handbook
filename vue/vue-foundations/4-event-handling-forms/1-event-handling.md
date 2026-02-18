---
source_course: "vue-foundations"
source_lesson: "vue-foundations-event-handling"
---

# Event Handling with v-on

Vue uses the `v-on` directive to listen to DOM events and execute JavaScript when they're triggered. This is how you make your applications interactive.

## Basic Event Listening

Use `v-on:event="handler"` to listen to events:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <p>Count: {{ count }}</p>
  <button v-on:click="increment">Click me</button>
</template>
```

## Shorthand: @

The `@` symbol is shorthand for `v-on:`:

```vue
<template>
  <!-- These are equivalent -->
  <button v-on:click="increment">Click</button>
  <button @click="increment">Click</button>
  
  <!-- @ is the standard convention -->
  <button @click="handleClick">Click</button>
  <input @input="handleInput" />
  <form @submit="handleSubmit">...</form>
</template>
```

## Inline Handlers

For simple operations, use inline expressions:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
const message = ref('')
</script>

<template>
  <!-- Inline expression -->
  <button @click="count++">Increment</button>
  <button @click="count--">Decrement</button>
  <button @click="count = 0">Reset</button>
  
  <!-- Inline function -->
  <button @click="() => count += 10">Add 10</button>
  
  <!-- Multiple statements -->
  <button @click="count++; message = 'Clicked!'">Click</button>
</template>
```

## Method Handlers

For complex logic, use methods:

```vue
<script setup>
import { ref } from 'vue'

const items = ref<string[]>([])

function addItem() {
  const item = `Item ${items.value.length + 1}`
  items.value.push(item)
}

function removeItem(index: number) {
  items.value.splice(index, 1)
}

function clearAll() {
  if (confirm('Remove all items?')) {
    items.value = []
  }
}
</script>

<template>
  <button @click="addItem">Add Item</button>
  <button @click="clearAll">Clear All</button>
  
  <ul>
    <li v-for="(item, index) in items" :key="index">
      {{ item }}
      <button @click="removeItem(index)">Remove</button>
    </li>
  </ul>
</template>
```

## Accessing the Event Object

Vue passes the native DOM event to handlers:

```vue
<script setup>
function handleClick(event: MouseEvent) {
  console.log('Button clicked!')
  console.log('Target:', event.target)
  console.log('Position:', event.clientX, event.clientY)
}

function handleInput(event: Event) {
  const target = event.target as HTMLInputElement
  console.log('Input value:', target.value)
}
</script>

<template>
  <button @click="handleClick">Click me</button>
  <input @input="handleInput" />
</template>
```

### Passing Arguments AND the Event

Use `$event` to pass the event along with other arguments:

```vue
<script setup>
function handleAction(action: string, event: MouseEvent) {
  console.log(`Action: ${action}`)
  console.log('Event:', event)
}

function handleItemClick(itemId: number, event: MouseEvent) {
  event.stopPropagation()
  console.log(`Clicked item ${itemId}`)
}
</script>

<template>
  <button @click="handleAction('save', $event)">Save</button>
  
  <div @click="console.log('Parent clicked')">
    <button @click="handleItemClick(42, $event)">Item 42</button>
  </div>
</template>
```

## Common Events

```vue
<template>
  <!-- Mouse events -->
  <button @click="onClick">Click</button>
  <button @dblclick="onDoubleClick">Double Click</button>
  <div @mouseenter="onMouseEnter">Hover me</div>
  <div @mouseleave="onMouseLeave">Hover me</div>
  
  <!-- Keyboard events -->
  <input @keydown="onKeyDown" />
  <input @keyup="onKeyUp" />
  <input @keypress="onKeyPress" />
  
  <!-- Form events -->
  <input @input="onInput" />
  <input @change="onChange" />
  <input @focus="onFocus" />
  <input @blur="onBlur" />
  <form @submit="onSubmit">...</form>
  
  <!-- Other events -->
  <div @scroll="onScroll">Scrollable content</div>
  <img @load="onImageLoad" />
  <img @error="onImageError" />
</template>
```

## Resources

- [Event Handling](https://vuejs.org/guide/essentials/event-handling.html) — Official Vue documentation on event handling

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*