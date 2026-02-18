---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-component-basics"
---

# Building with Components

Components are the building blocks of Svelte applications. Each component is a self-contained unit with its own markup, styles, and logic.

## Creating a Component

Every `.svelte` file is a component. Let's create a simple `Button.svelte`:

```svelte
<!-- Button.svelte -->
<button>
  Click me
</button>

<style>
  button {
    background: #ff3e00;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
  }
</style>
```

## Using a Component

To use a component, you **import** it and use it like an HTML tag:

```svelte
<!-- App.svelte -->
<script>
  import Button from './Button.svelte';
</script>

<h1>My App</h1>
<Button />
<Button />
<Button />
```

Each `<Button />` creates a separate **instance** of the component. They're independent — changing one doesn't affect the others.

## Component File Organization

A common pattern is to organize components in a `lib` or `components` folder:

```
src/
├── routes/
│   └── +page.svelte
└── lib/
    └── components/
        ├── Button.svelte
        ├── Card.svelte
        └── Header.svelte
```

In SvelteKit, you can import from `$lib`:

```svelte
<script>
  import Button from '$lib/components/Button.svelte';
</script>
```

## Component Names

- Component files should be **PascalCase** (e.g., `MyButton.svelte`)
- This distinguishes them from regular HTML elements
- Use them as `<MyButton />`, not `<my-button />`

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*