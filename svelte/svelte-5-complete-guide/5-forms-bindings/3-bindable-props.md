---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-bindable-props"
---

# Two-Way Binding for Component Props

Sometimes you want a parent component to bind to a child's prop — like how you bind to an input's value. The `$bindable` rune makes this possible.

## The Problem

By default, props flow one way (parent → child). What if you want changes in the child to flow back up?

## Without $bindable

You'd need callback props:

```svelte
<!-- Child.svelte -->
<script>
  let { value, onchange } = $props();
</script>

<input value={value} oninput={(e) => onchange(e.target.value)} />
```

```svelte
<!-- Parent.svelte -->
<script>
  let text = $state('');
</script>

<Child value={text} onchange={(v) => text = v} />
```

This works, but it's verbose.

## With $bindable

```svelte
<!-- FancyInput.svelte -->
<script>
  let { value = $bindable('') } = $props();
</script>

<input bind:value={value} class="fancy" />

<style>
  .fancy {
    border: 2px solid purple;
    padding: 8px;
    border-radius: 8px;
  }
</style>
```

```svelte
<!-- Parent.svelte -->
<script>
  import FancyInput from './FancyInput.svelte';
  
  let username = $state('');
</script>

<FancyInput bind:value={username} />
<p>Username: {username}</p>
```

Now changes in FancyInput automatically update `username` in the parent!

## How $bindable Works

1. In the child, mark the prop with `$bindable(defaultValue)`
2. In the parent, use `bind:propName={variable}`
3. Changes flow both directions automatically

## When to Use $bindable

Common use cases:
- Custom input components (text fields, sliders, toggles)
- Modal open/close state
- Accordion expanded state
- Any reusable component that controls a value

## Important Notes

- `$bindable` provides a default value if the parent doesn't bind
- If the parent passes the prop normally (without `bind:`), it acts as a regular prop
- The child can modify `$bindable` props directly

## Resources

- [$bindable Documentation](https://svelte.dev/docs/svelte/$bindable) — Official guide to creating bindable props.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*