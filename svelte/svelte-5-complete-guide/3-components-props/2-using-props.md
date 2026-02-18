---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-using-props"
---

# The $props Rune

Components become truly powerful when they can receive data from their parent. In Svelte 5, we use the `$props` rune to declare what data a component accepts.

## Declaring Props

```svelte
<!-- Greeting.svelte -->
<script>
  let { name } = $props();
</script>

<h1>Hello, {name}!</h1>
```

`$props()` returns an object containing all the props passed to this component. We destructure it to get the values we need.

## Passing Props

Pass props just like HTML attributes:

```svelte
<!-- App.svelte -->
<script>
  import Greeting from './Greeting.svelte';
</script>

<Greeting name="Alice" />
<Greeting name="Bob" />
```

## Default Values

Provide defaults using JavaScript's default parameter syntax:

```svelte
<script>
  let { 
    name = 'World',     // Default: 'World'
    age = 18,           // Default: 18
    isActive = false    // Default: false
  } = $props();
</script>

<p>{name} is {age} years old</p>
```

## Multiple Props

```svelte
<!-- UserCard.svelte -->
<script>
  let { 
    name,
    email,
    avatar = '/default-avatar.png',
    role = 'user'
  } = $props();
</script>

<div class="card">
  <img src={avatar} alt={name} />
  <h2>{name}</h2>
  <p>{email}</p>
  <span class="badge">{role}</span>
</div>
```

## Rest Props

Capture remaining props with the spread operator:

```svelte
<!-- Button.svelte -->
<script>
  let { children, variant = 'primary', ...rest } = $props();
</script>

<!-- All extra props (like id, class, onclick) are passed to the button -->
<button class="btn btn-{variant}" {...rest}>
  {@render children()}
</button>
```

Now you can use it like:

```svelte
<Button id="submit-btn" class="large" onclick={handleSubmit}>
  Submit
</Button>
```

## Dynamic Props

Pass reactive values as props:

```svelte
<script>
  import Counter from './Counter.svelte';
  
  let startValue = $state(10);
</script>

<input type="number" bind:value={startValue} />
<Counter initial={startValue} />
```

## Resources

- [$props Documentation](https://svelte.dev/docs/svelte/$props) — Official guide to component props in Svelte 5.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*