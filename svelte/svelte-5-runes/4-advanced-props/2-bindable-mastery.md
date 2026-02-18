---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-bindable-mastery"
---

# Two-Way Binding for Components

The `$bindable` rune allows parent components to bind to child props, enabling two-way data flow.

## Basic $bindable

```svelte
<!-- Toggle.svelte -->
<script>
  let { on = $bindable(false) } = $props();
</script>

<button onclick={() => on = !on}>
  {on ? 'ON' : 'OFF'}
</button>
```

Parent can bind:

```svelte
<script>
  let lightOn = $state(false);
</script>

<Toggle bind:on={lightOn} />
<p>Light is {lightOn ? 'on' : 'off'}</p>
```

## Multiple Bindable Props

```svelte
<!-- Slider.svelte -->
<script>
  let {
    value = $bindable(50),
    min = $bindable(0),
    max = $bindable(100)
  } = $props();
</script>

<input 
  type="range" 
  bind:value 
  {min} 
  {max}
/>
```

```svelte
<Slider bind:value={volume} bind:max={maxVolume} />
```

## Bindable with Fallback

The value passed to `$bindable()` is the fallback when no binding is provided:

```svelte
<script>
  let { value = $bindable('default text') } = $props();
</script>

<!-- If parent doesn't bind, value starts as 'default text' -->
```

## Real-World Example: Custom Input

```svelte
<!-- EmailInput.svelte -->
<script>
  let { 
    value = $bindable(''),
    valid = $bindable(false),
    ...rest 
  } = $props();
  
  // Validate email format
  $effect(() => {
    valid = /^[^@]+@[^@]+\.[^@]+$/.test(value);
  });
</script>

<div class="email-input" class:invalid={value && !valid}>
  <input 
    type="email" 
    bind:value
    {...rest}
  />
  {#if value && !valid}
    <span class="error">Invalid email</span>
  {/if}
</div>
```

Parent:

```svelte
<script>
  let email = $state('');
  let emailValid = $state(false);
</script>

<EmailInput bind:value={email} bind:valid={emailValid} />

<button disabled={!emailValid}>
  Submit
</button>
```

## When to Use $bindable vs Callback Props

| Use Case | Approach |
|----------|----------|
| Form inputs (text, select) | `$bindable` |
| Toggle states (open/close) | `$bindable` |
| Events (click, submit) | Callback props |
| One-time actions (delete, save) | Callback props |
| Parent needs to know about changes | Either works |

```svelte
<!-- Bindable: parent can sync state -->
<Modal bind:open={showModal} />

<!-- Callback: parent handles the action -->
<Modal onclose={() => showModal = false} />
```

## Resources

- [$bindable Documentation](https://svelte.dev/docs/svelte/$bindable) — Official guide to creating bindable props.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*