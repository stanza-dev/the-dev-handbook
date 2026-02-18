---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-anatomy-of-a-component"
---

# Understanding .svelte Files

Svelte applications are built from **components**. Each component lives in a `.svelte` file that combines HTML, CSS, and JavaScript in a single, cohesive unit.

## The Three Sections

Every Svelte component can have up to three sections:

### 1. The `<script>` Block

This is where your JavaScript logic lives. You can import other components, declare variables, and define functions.

```svelte
<script>
  // JavaScript goes here
  let name = 'World';
  
  function greet() {
    alert(`Hello ${name}!`);
  }
</script>
```

### 2. The Markup (Template)

This is your HTML. You can use JavaScript expressions inside curly braces `{}`.

```svelte
<h1>Hello {name}!</h1>
<button onclick={greet}>Click me</button>
```

### 3. The `<style>` Block

This is your CSS. The magic? **Styles are scoped by default** — they only affect elements in THIS component.

```svelte
<style>
  h1 {
    color: #ff3e00;
    font-family: 'Comic Sans MS';
  }
</style>
```

## Complete Example

Here's a complete component with all three sections:

```svelte
<script>
  let count = 0;
  
  function increment() {
    count += 1;
  }
</script>

<button onclick={increment}>
  Clicked {count} times
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
  
  button:hover {
    background: #cc3200;
  }
</style>
```

## Key Points to Remember

| Section | Required? | Purpose |
|---------|-----------|--------|
| `<script>` | No | Logic, state, imports |
| Markup | Yes | The visible HTML |
| `<style>` | No | Scoped CSS styling |

**Important**: The order doesn't matter! You can put `<style>` at the top if you prefer. However, the convention is script → markup → style.

## Why Scoped Styles Matter

In traditional web development, CSS is global — a style in one file can accidentally affect elements everywhere. In Svelte, each component's styles are isolated:

```svelte
<!-- Header.svelte -->
<style>
  p { color: red; }  /* Only affects <p> in THIS file */
</style>
<p>I'm red</p>

<!-- Footer.svelte -->
<p>I'm NOT red — different component!</p>
```

Svelte achieves this by adding unique class names to your elements during compilation.

## Resources

- [.svelte Files](https://svelte.dev/docs/svelte/svelte-files) — Official documentation on the structure of .svelte files.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*