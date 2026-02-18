---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-text-inputs"
---

# Understanding Two-Way Binding

Normally, data in Svelte flows **one way**: from parent to child via props. But form inputs are special — we often want changes in the input to update our state immediately.

Svelte's `bind:` directive creates **two-way binding**: the state updates the input, and the input updates the state.

## Basic Text Input

```svelte
<script>
  let name = $state('');
</script>

<input bind:value={name} />
<p>Hello, {name || 'stranger'}!</p>
```

As you type, `name` updates in real-time. If you change `name` programmatically, the input updates too.

## Without bind: (One-Way)

To understand the difference, here's one-way binding:

```svelte
<script>
  let name = $state('');
</script>

<!-- This only displays the value, typing doesn't update `name` -->
<input value={name} />

<!-- You'd need to manually handle the input event -->
<input value={name} oninput={(e) => name = e.target.value} />
```

`bind:value` does this for you automatically!

## Textarea

Works exactly like text inputs:

```svelte
<script>
  let bio = $state('');
</script>

<textarea bind:value={bio} rows="5"></textarea>
<p>Character count: {bio.length}</p>
```

## Number Inputs

Svelte automatically converts to numbers when using `type="number"`:

```svelte
<script>
  let age = $state(25);
  let quantity = $state(1);
</script>

<input type="number" bind:value={age} />
<input type="range" bind:value={quantity} min="1" max="10" />

<p>age is a {typeof age}</p>  <!-- "number", not "string"! -->
```

## Common Form Elements

```svelte
<script>
  let email = $state('');
  let password = $state('');
  let search = $state('');
</script>

<input type="email" bind:value={email} placeholder="Email" />
<input type="password" bind:value={password} placeholder="Password" />
<input type="search" bind:value={search} placeholder="Search..." />
```

## Resources

- [bind: Directive](https://svelte.dev/docs/svelte/bind) — Complete guide to Svelte's binding system.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*