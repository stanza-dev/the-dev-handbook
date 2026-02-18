---
source_course: "vue-foundations"
source_lesson: "vue-foundations-form-input-bindings"
---

# Form Input Bindings with v-model

The `v-model` directive creates **two-way data binding** on form inputs. It automatically syncs the input's value with your component's state.

## Basic Usage

Without `v-model`, you'd need to handle both directions manually:

```vue
<script setup>
import { ref } from 'vue'

const message = ref('')

function handleInput(event: Event) {
  message.value = (event.target as HTMLInputElement).value
}
</script>

<template>
  <!-- Manual two-way binding -->
  <input :value="message" @input="handleInput" />
</template>
```

With `v-model`, it's much simpler:

```vue
<script setup>
import { ref } from 'vue'

const message = ref('')
</script>

<template>
  <!-- Two-way binding with v-model -->
  <input v-model="message" />
  <p>You typed: {{ message }}</p>
</template>
```

## Text Inputs

```vue
<script setup>
import { ref } from 'vue'

const username = ref('')
const bio = ref('')
</script>

<template>
  <!-- Single line input -->
  <input v-model="username" placeholder="Username" />
  
  <!-- Textarea for multiline -->
  <textarea v-model="bio" placeholder="Tell us about yourself"></textarea>
</template>
```

**Note**: Don't use `{{ }}` inside textarea—it won't work:

```vue
<!-- ❌ Wrong -->
<textarea>{{ message }}</textarea>

<!-- ✅ Correct -->
<textarea v-model="message"></textarea>
```

## Checkboxes

### Single Checkbox (Boolean)

```vue
<script setup>
import { ref } from 'vue'

const isAgreed = ref(false)
const receiveNewsletter = ref(true)
</script>

<template>
  <label>
    <input type="checkbox" v-model="isAgreed" />
    I agree to the terms
  </label>
  <p>Agreed: {{ isAgreed }}</p>
  
  <label>
    <input type="checkbox" v-model="receiveNewsletter" />
    Subscribe to newsletter
  </label>
</template>
```

### Multiple Checkboxes (Array)

```vue
<script setup>
import { ref } from 'vue'

const selectedFruits = ref<string[]>([])
</script>

<template>
  <label>
    <input type="checkbox" value="apple" v-model="selectedFruits" />
    Apple
  </label>
  <label>
    <input type="checkbox" value="banana" v-model="selectedFruits" />
    Banana
  </label>
  <label>
    <input type="checkbox" value="cherry" v-model="selectedFruits" />
    Cherry
  </label>
  
  <p>Selected: {{ selectedFruits.join(', ') }}</p>
</template>
```

## Radio Buttons

```vue
<script setup>
import { ref } from 'vue'

const selectedSize = ref('medium')
const paymentMethod = ref('')
</script>

<template>
  <div>
    <label>
      <input type="radio" value="small" v-model="selectedSize" />
      Small
    </label>
    <label>
      <input type="radio" value="medium" v-model="selectedSize" />
      Medium
    </label>
    <label>
      <input type="radio" value="large" v-model="selectedSize" />
      Large
    </label>
    <p>Selected size: {{ selectedSize }}</p>
  </div>
</template>
```

## Select Dropdowns

### Single Select

```vue
<script setup>
import { ref } from 'vue'

const selectedCountry = ref('')
</script>

<template>
  <select v-model="selectedCountry">
    <option value="" disabled>Select a country</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
    <option value="au">Australia</option>
  </select>
  <p>Selected: {{ selectedCountry }}</p>
</template>
```

### Multiple Select

```vue
<script setup>
import { ref } from 'vue'

const selectedSkills = ref<string[]>([])
</script>

<template>
  <select v-model="selectedSkills" multiple>
    <option value="javascript">JavaScript</option>
    <option value="typescript">TypeScript</option>
    <option value="vue">Vue.js</option>
    <option value="react">React</option>
  </select>
  <p>Selected: {{ selectedSkills.join(', ') }}</p>
</template>
```

### Dynamic Options

```vue
<script setup>
import { ref } from 'vue'

const categories = ref([
  { id: 1, name: 'Electronics' },
  { id: 2, name: 'Clothing' },
  { id: 3, name: 'Books' }
])

const selectedCategory = ref(null)
</script>

<template>
  <select v-model="selectedCategory">
    <option :value="null" disabled>Choose a category</option>
    <option 
      v-for="category in categories" 
      :key="category.id"
      :value="category.id"
    >
      {{ category.name }}
    </option>
  </select>
</template>
```

## v-model Modifiers

### .lazy - Sync on Change (Not Input)

```vue
<template>
  <!-- Updates on every keystroke (default) -->
  <input v-model="message" />
  
  <!-- Updates only when input loses focus or Enter is pressed -->
  <input v-model.lazy="message" />
</template>
```

### .number - Cast to Number

```vue
<script setup>
import { ref } from 'vue'

const age = ref(0)  // Will be a number, not string
const quantity = ref(1)
</script>

<template>
  <!-- Without .number, age would be "25" (string) -->
  <input v-model.number="age" type="number" />
  <p>Age + 1 = {{ age + 1 }}</p>
  
  <input v-model.number="quantity" type="text" />
</template>
```

### .trim - Remove Whitespace

```vue
<script setup>
import { ref } from 'vue'

const username = ref('')
</script>

<template>
  <!-- Automatically trims leading/trailing whitespace -->
  <input v-model.trim="username" />
  <p>Username: "{{ username }}"</p>
</template>
```

### Combining Modifiers

```vue
<template>
  <input v-model.lazy.trim="email" type="email" />
  <input v-model.number.lazy="price" type="text" />
</template>
```

## Resources

- [Form Input Bindings](https://vuejs.org/guide/essentials/forms.html) — Official Vue documentation on v-model and form handling

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*