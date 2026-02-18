---
source_course: "vue-foundations"
source_lesson: "vue-foundations-text-interpolation"
---

# Text Interpolation and Expressions

Vue's template syntax lets you declaratively bind the rendered DOM to the underlying component's data. At the heart of this is **text interpolation**—the most basic form of data binding.

## The Mustache Syntax

The most basic form of data binding is text interpolation using double curly braces, often called "mustaches":

```vue
<script setup>
import { ref } from 'vue'

const message = ref('Hello, Vue!')
const username = ref('Alice')
</script>

<template>
  <p>{{ message }}</p>
  <p>Welcome, {{ username }}!</p>
</template>
```

The mustache tag is replaced with the value of the corresponding property. It updates automatically whenever that property changes.

## JavaScript Expressions

Vue supports full JavaScript expressions inside mustaches:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(10)
const price = ref(29.99)
const firstName = ref('John')
const lastName = ref('Doe')
const isActive = ref(true)
const items = ref(['Apple', 'Banana', 'Cherry'])
</script>

<template>
  <!-- Arithmetic -->
  <p>Count + 5 = {{ count + 5 }}</p>
  <p>Total: ${{ (price * count).toFixed(2) }}</p>
  
  <!-- String manipulation -->
  <p>Full name: {{ firstName + ' ' + lastName }}</p>
  <p>Uppercase: {{ firstName.toUpperCase() }}</p>
  
  <!-- Ternary operator -->
  <p>Status: {{ isActive ? 'Active' : 'Inactive' }}</p>
  
  <!-- Method calls -->
  <p>Items: {{ items.join(', ') }}</p>
  <p>Reversed: {{ message.split('').reverse().join('') }}</p>
</template>
```

## Expression Limitations

Each binding can only contain **one single expression**. The following will **NOT** work:

```vue
<!-- This is a statement, not an expression -->
{{ let x = 1 }}

<!-- Flow control won't work, use ternary expressions -->
{{ if (ok) { return message } }}

<!-- Multiple expressions -->
{{ message; count++ }}
```

### What's an Expression vs Statement?

- **Expression**: Produces a value (`count + 1`, `ok ? 'yes' : 'no'`)
- **Statement**: Performs an action (`if`, `for`, `let x = 1`)

## Calling Functions

You can call component methods inside interpolations:

```vue
<script setup>
import { ref } from 'vue'

const birthday = ref('1990-05-15')

function formatDate(dateString: string): string {
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

function calculateAge(dateString: string): number {
  const birth = new Date(dateString)
  const today = new Date()
  let age = today.getFullYear() - birth.getFullYear()
  const monthDiff = today.getMonth() - birth.getMonth()
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
    age--
  }
  return age
}
</script>

<template>
  <p>Birthday: {{ formatDate(birthday) }}</p>
  <p>Age: {{ calculateAge(birthday) }} years old</p>
</template>
```

**Important**: Functions called inside templates will be called every time the component re-renders. They should not have side effects (like changing data or triggering async operations).

## Raw HTML with v-html

Mustaches interpret data as plain text. To render actual HTML, use the `v-html` directive:

```vue
<script setup>
import { ref } from 'vue'

const rawHtml = ref('<span style="color: red">This is red</span>')
const description = ref('<strong>Bold</strong> and <em>italic</em> text')
</script>

<template>
  <!-- Renders as plain text: <span style="color: red">... -->
  <p>{{ rawHtml }}</p>
  
  <!-- Renders as actual HTML -->
  <p v-html="rawHtml"></p>
  <p v-html="description"></p>
</template>
```

**Security Warning**: Rendering arbitrary HTML can lead to XSS (Cross-Site Scripting) attacks. Only use `v-html` on trusted content, **never** on user-provided content.

## Template Expressions Have Limited Access

Template expressions are sandboxed and only have access to:

- Component properties and methods
- A [restricted list of globals](https://github.com/vuejs/core/blob/main/packages/shared/src/globalsAllowList.ts) like `Math`, `Date`, `Array`, etc.

You can add custom globals via `app.config.globalProperties`, but this is rarely needed.

## Resources

- [Template Syntax](https://vuejs.org/guide/essentials/template-syntax.html) — Official Vue documentation on template syntax and text interpolation

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*