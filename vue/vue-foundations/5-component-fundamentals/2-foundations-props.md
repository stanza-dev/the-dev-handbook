---
source_course: "vue-foundations"
source_lesson: "vue-foundations-props"
---

# Passing Data with Props

Props are custom attributes you can register on a component. They let parent components pass data to child components.

## Defining Props

Use `defineProps()` to declare what props a component accepts:

```vue
<!-- src/components/GreetingCard.vue -->
<script setup lang="ts">
const props = defineProps<{
  name: string
  age?: number  // Optional prop
}>()
</script>

<template>
  <div class="greeting">
    <h2>Hello, {{ name }}!</h2>
    <p v-if="age">You are {{ age }} years old.</p>
  </div>
</template>
```

## Passing Props

Pass props like HTML attributes:

```vue
<!-- Parent component -->
<script setup lang="ts">
import GreetingCard from './components/GreetingCard.vue'
import { ref } from 'vue'

const userName = ref('Alice')
const userAge = ref(25)
</script>

<template>
  <!-- Static props -->
  <GreetingCard name="Bob" />
  <GreetingCard name="Charlie" :age="30" />
  
  <!-- Dynamic props with v-bind -->
  <GreetingCard :name="userName" :age="userAge" />
</template>
```

## Prop Types

### With TypeScript (Recommended)

```vue
<script setup lang="ts">
type Status = 'pending' | 'active' | 'completed'

type User = {
  id: number
  name: string
  email: string
}

const props = defineProps<{
  // Required string
  title: string
  
  // Optional number
  count?: number
  
  // Array of strings
  tags: string[]
  
  // Object type
  user: User
  
  // Union type
  status: Status
  
  // Boolean
  isActive: boolean
  
  // Function
  onUpdate: (value: string) => void
}>()
</script>
```

### With Default Values

Use `withDefaults()` for default values:

```vue
<script setup lang="ts">
type Props = {
  title: string
  count?: number
  items?: string[]
  theme?: 'light' | 'dark'
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
  items: () => [],  // Use factory for objects/arrays
  theme: 'light'
})
</script>
```

## Using Props in the Template

Props are directly available in templates:

```vue
<script setup lang="ts">
const props = defineProps<{
  title: string
  items: string[]
  showHeader: boolean
}>()
</script>

<template>
  <div class="container">
    <h1 v-if="showHeader">{{ title }}</h1>
    <ul>
      <li v-for="item in items" :key="item">{{ item }}</li>
    </ul>
  </div>
</template>
```

## Props are Read-Only!

Never mutate props directly—they flow one-way from parent to child:

```vue
<script setup lang="ts">
const props = defineProps<{
  count: number
}>()

// ❌ NEVER do this - will cause warnings and bugs
function increment() {
  props.count++  // Error: cannot mutate props
}

// ✅ Instead, emit an event to the parent
const emit = defineEmits<{
  update: [newCount: number]
}>()

function increment() {
  emit('update', props.count + 1)
}
</script>
```

## Practical Example: Product Card

```vue
<!-- src/components/ProductCard.vue -->
<script setup lang="ts">
type Product = {
  id: number
  name: string
  price: number
  image: string
  inStock: boolean
}

const props = withDefaults(defineProps<{
  product: Product
  showActions?: boolean
}>(), {
  showActions: true
})

const emit = defineEmits<{
  addToCart: [productId: number]
}>()

function formatPrice(price: number): string {
  return `$${price.toFixed(2)}`
}
</script>

<template>
  <div class="product-card">
    <img :src="product.image" :alt="product.name" />
    
    <div class="content">
      <h3>{{ product.name }}</h3>
      <p class="price">{{ formatPrice(product.price) }}</p>
      
      <span :class="['stock', { 'in-stock': product.inStock }]">
        {{ product.inStock ? 'In Stock' : 'Out of Stock' }}
      </span>
    </div>
    
    <div v-if="showActions" class="actions">
      <button 
        @click="emit('addToCart', product.id)"
        :disabled="!product.inStock"
      >
        Add to Cart
      </button>
    </div>
  </div>
</template>

<style scoped>
.product-card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
}

.product-card img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.content {
  padding: 1rem;
}

.price {
  font-size: 1.25rem;
  font-weight: bold;
  color: #42b883;
}

.stock {
  font-size: 0.875rem;
  color: #e53e3e;
}

.stock.in-stock {
  color: #38a169;
}

.actions {
  padding: 1rem;
  border-top: 1px solid #e0e0e0;
}
</style>
```

```vue
<!-- Using the ProductCard -->
<script setup lang="ts">
import ProductCard from './components/ProductCard.vue'
import { ref } from 'vue'

const products = ref([
  { id: 1, name: 'Laptop', price: 999.99, image: '/laptop.jpg', inStock: true },
  { id: 2, name: 'Phone', price: 699.99, image: '/phone.jpg', inStock: false },
])

function handleAddToCart(productId: number) {
  console.log('Adding product to cart:', productId)
}
</script>

<template>
  <div class="products">
    <ProductCard
      v-for="product in products"
      :key="product.id"
      :product="product"
      @add-to-cart="handleAddToCart"
    />
  </div>
</template>
```

## Resources

- [Props](https://vuejs.org/guide/components/props.html) — Official Vue documentation on component props

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*