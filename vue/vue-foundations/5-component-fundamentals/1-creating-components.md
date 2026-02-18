---
source_course: "vue-foundations"
source_lesson: "vue-foundations-creating-components"
---

# Creating and Using Components

Components are the building blocks of Vue applications. They're reusable, self-contained pieces of UI that encapsulate their own template, logic, and styling.

## Why Components?

Components let you:

- **Reuse UI elements** across your application
- **Organize code** into manageable pieces
- **Encapsulate complexity** behind simple interfaces
- **Test in isolation** for reliability

## Creating a Component

Create a new `.vue` file for each component:

```vue
<!-- src/components/WelcomeMessage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const isVisible = ref(true)
</script>

<template>
  <div v-if="isVisible" class="welcome">
    <h2>Welcome to Vue!</h2>
    <p>You've successfully created a component.</p>
    <button @click="isVisible = false">Dismiss</button>
  </div>
</template>

<style scoped>
.welcome {
  background: #e8f5e9;
  padding: 1rem;
  border-radius: 8px;
  border-left: 4px solid #42b883;
}

h2 {
  margin: 0 0 0.5rem;
  color: #2e7d32;
}

button {
  margin-top: 0.5rem;
}
</style>
```

## Using a Component

Import and use components in other components:

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import WelcomeMessage from './components/WelcomeMessage.vue'
import UserProfile from './components/UserProfile.vue'
import NavBar from './components/NavBar.vue'
</script>

<template>
  <div class="app">
    <NavBar />
    <main>
      <WelcomeMessage />
      <UserProfile />
    </main>
  </div>
</template>
```

With `<script setup>`, imported components are **automatically available** in the template. No manual registration needed!

## Component Naming Conventions

### File Names

Use **PascalCase** for component files:

```
✅ UserProfile.vue
✅ NavigationBar.vue
✅ TodoListItem.vue

❌ userProfile.vue
❌ user-profile.vue
```

### In Templates

You can use either PascalCase or kebab-case:

```vue
<template>
  <!-- Both are valid -->
  <UserProfile />
  <user-profile />
  
  <!-- PascalCase is recommended for clarity -->
  <NavigationBar />
  <TodoListItem />
</template>
```

### Multi-Word Names

Always use multi-word names to avoid conflicts with HTML elements:

```vue
<!-- ❌ Bad - might conflict with future HTML elements -->
<Header />
<Footer />
<Button />

<!-- ✅ Good - clearly custom components -->
<AppHeader />
<AppFooter />
<BaseButton />
```

## Component Organization

Organize components in a logical structure:

```
src/
├── components/
│   ├── common/           # Shared, generic components
│   │   ├── BaseButton.vue
│   │   ├── BaseInput.vue
│   │   └── BaseCard.vue
│   ├── layout/           # Layout components
│   │   ├── AppHeader.vue
│   │   ├── AppFooter.vue
│   │   └── AppSidebar.vue
│   └── features/         # Feature-specific components
│       ├── UserProfile.vue
│       ├── ProductCard.vue
│       └── CommentList.vue
├── views/                # Page-level components
│   ├── HomeView.vue
│   ├── AboutView.vue
│   └── UserView.vue
└── App.vue               # Root component
```

## Self-Closing Tags

Components without content should use self-closing tags:

```vue
<template>
  <!-- ✅ Good - self-closing -->
  <UserAvatar />
  <LoadingSpinner />
  
  <!-- ❌ Unnecessary closing tag -->
  <UserAvatar></UserAvatar>
</template>
```

## Multiple Components Example

Let's build a simple card system:

```vue
<!-- src/components/BaseCard.vue -->
<script setup lang="ts">
// This component will accept props later
</script>

<template>
  <div class="card">
    <slot></slot>
  </div>
</template>

<style scoped>
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  margin-bottom: 1rem;
}
</style>
```

```vue
<!-- src/components/UserCard.vue -->
<script setup lang="ts">
import BaseCard from './BaseCard.vue'
</script>

<template>
  <BaseCard>
    <h3>John Doe</h3>
    <p>Software Developer</p>
  </BaseCard>
</template>
```

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import UserCard from './components/UserCard.vue'
</script>

<template>
  <div class="app">
    <h1>Team Members</h1>
    <UserCard />
    <UserCard />
    <UserCard />
  </div>
</template>
```

## Resources

- [Components Basics](https://vuejs.org/guide/essentials/component-basics.html) — Official Vue documentation on component fundamentals

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*