---
source_course: "vue-foundations"
source_lesson: "vue-foundations-attribute-bindings"
---

# Attribute Bindings with v-bind

Mustaches cannot be used inside HTML attributes. Instead, use the `v-bind` directive to dynamically bind attributes to JavaScript expressions.

## Basic Attribute Binding

The `v-bind` directive binds an attribute to a dynamic value:

```vue
<script setup>
import { ref } from 'vue'

const imageUrl = ref('/images/logo.png')
const linkUrl = ref('https://vuejs.org')
const inputId = ref('email-input')
</script>

<template>
  <!-- Bind the src attribute -->
  <img v-bind:src="imageUrl" alt="Logo" />
  
  <!-- Bind the href attribute -->
  <a v-bind:href="linkUrl">Vue.js Website</a>
  
  <!-- Bind the id attribute -->
  <input v-bind:id="inputId" type="email" />
  <label v-bind:for="inputId">Email:</label>
</template>
```

## Shorthand Syntax: `:`

Because `v-bind` is so common, Vue provides a shorthand—just use `:` before the attribute:

```vue
<template>
  <!-- Full syntax -->
  <img v-bind:src="imageUrl" />
  
  <!-- Shorthand (recommended) -->
  <img :src="imageUrl" />
  <a :href="linkUrl">Link</a>
  <input :id="inputId" />
</template>
```

The shorthand is the convention you'll see in most Vue code.

## Same-name Shorthand (Vue 3.4+)

When the attribute name matches the JavaScript variable name, you can use an even shorter syntax:

```vue
<script setup>
import { ref } from 'vue'

const id = ref('my-element')
const src = ref('/images/photo.jpg')
const href = ref('https://example.com')
</script>

<template>
  <!-- Same as :id="id" -->
  <div :id>Content</div>
  
  <!-- Same as :src="src" -->
  <img :src alt="Photo" />
  
  <!-- Same as :href="href" -->
  <a :href>Link</a>
</template>
```

## Boolean Attributes

Boolean attributes indicate true/false values by their presence. `v-bind` handles them specially:

```vue
<script setup>
import { ref } from 'vue'

const isDisabled = ref(true)
const isRequired = ref(false)
const isChecked = ref(true)
</script>

<template>
  <!-- Renders: <button disabled>...</button> -->
  <button :disabled="isDisabled">Can't click me</button>
  
  <!-- Renders: <input> (no required attribute) -->
  <input :required="isRequired" />
  
  <!-- Renders: <input type="checkbox" checked> -->
  <input type="checkbox" :checked="isChecked" />
</template>
```

Rules for boolean attributes:
- **Truthy value** (including `""` empty string): Attribute is included
- **Falsy value** (`false`, `null`, `undefined`): Attribute is omitted

## Binding Multiple Attributes

You can bind multiple attributes at once using an object:

```vue
<script setup>
import { ref, reactive } from 'vue'

const inputAttrs = reactive({
  id: 'email-field',
  type: 'email',
  placeholder: 'Enter your email',
  required: true
})

const imageAttrs = ref({
  src: '/images/avatar.jpg',
  alt: 'User avatar',
  width: 100,
  height: 100
})
</script>

<template>
  <!-- Bind all attributes from the object -->
  <input v-bind="inputAttrs" />
  <!-- Renders: <input id="email-field" type="email" placeholder="Enter your email" required> -->
  
  <img v-bind="imageAttrs" />
  <!-- Renders: <img src="/images/avatar.jpg" alt="User avatar" width="100" height="100"> -->
</template>
```

## Dynamic Attribute Names

You can also dynamically bind attribute names using bracket syntax:

```vue
<script setup>
import { ref } from 'vue'

const attributeName = ref('title')
const attributeValue = ref('This is a tooltip')
</script>

<template>
  <!-- Dynamic attribute name -->
  <div :[attributeName]="attributeValue">
    Hover over me!
  </div>
  <!-- Renders: <div title="This is a tooltip">...</div> -->
</template>
```

## Common Patterns

### Conditional Classes and Styles

```vue
<script setup>
import { ref } from 'vue'

const isActive = ref(true)
const hasError = ref(false)
const fontSize = ref(16)
</script>

<template>
  <!-- Binding to expressions -->
  <div :class="isActive ? 'active' : 'inactive'">Status</div>
  
  <!-- Dynamic styles -->
  <p :style="{ fontSize: fontSize + 'px' }">Styled text</p>
</template>
```

### Image Sources

```vue
<script setup>
import { ref, computed } from 'vue'

const userId = ref(42)
const avatarUrl = computed(() => `/api/users/${userId.value}/avatar`)
</script>

<template>
  <img :src="avatarUrl" :alt="`User ${userId} avatar`" />
</template>
```

### Aria Attributes for Accessibility

```vue
<script setup>
import { ref } from 'vue'

const isExpanded = ref(false)
const menuId = ref('main-menu')
</script>

<template>
  <button 
    :aria-expanded="isExpanded" 
    :aria-controls="menuId"
    @click="isExpanded = !isExpanded"
  >
    Toggle Menu
  </button>
  <nav :id="menuId" v-show="isExpanded">
    <!-- Menu items -->
  </nav>
</template>
```

## Resources

- [Attribute Bindings](https://vuejs.org/guide/essentials/template-syntax.html#attribute-bindings) — Official documentation on v-bind and attribute bindings

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*