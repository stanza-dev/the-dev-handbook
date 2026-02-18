---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-why-snippets"
---

# The Evolution from Slots to Snippets

In Svelte 4 and earlier, components passed content using `<slot>` elements. While familiar, slots had significant limitations.

## Slots in Svelte 4

```svelte
<!-- Card.svelte (Svelte 4) -->
<div class="card">
  <slot name="header">Default Header</slot>
  <slot />  <!-- Default slot -->
  <slot name="footer" />
</div>
```

```svelte
<!-- Usage -->
<Card>
  <h2 slot="header">My Title</h2>
  <p>Main content goes here</p>
  <button slot="footer">Save</button>
</Card>
```

## The Problems with Slots

### 1. Awkward Slot Props

```svelte
<!-- Child passes data up via slot props -->
<slot item={currentItem} />

<!-- Parent receives it with weird syntax -->
<List let:item>
  <p>{item.name}</p>
</List>
```

The `let:` syntax was non-obvious and confusing.

### 2. Component Boundary Restriction

Slots only worked at component boundaries — you couldn't reuse markup within a single component.

### 3. Type Safety Issues

Typing slot props was difficult and error-prone.

## Enter Snippets

Snippets are **reusable blocks of markup** that work like functions:

```svelte
{#snippet greeting(name)}
  <h1>Hello, {name}!</h1>
{/snippet}

{@render greeting('Alice')}
{@render greeting('Bob')}
```

## Key Differences

| Feature | Slots (Svelte 4) | Snippets (Svelte 5) |
|---------|------------------|---------------------|
| Reusability | Only at component boundary | Anywhere in template |
| Parameters | `let:prop` syntax | Function parameters |
| Type Safety | Limited | Full TypeScript support |
| Recursion | Needed `<svelte:self>` | Just call the snippet |
| Default Content | `<slot>fallback</slot>` | Check if snippet exists |

## The Syntax

```svelte
{#snippet name(param1, param2)}
  <!-- Template content -->
{/snippet}

{@render name(arg1, arg2)}
```

Snippets are defined with `{#snippet}` and rendered with `{@render}`.

## Resources

- [{#snippet} Documentation](https://svelte.dev/docs/svelte/snippet) — Official guide to Svelte 5 snippets.

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*