---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-conditionals"
---

# Showing and Hiding Content

Real applications need to show different content based on conditions. Svelte's `{#if}` block makes this easy and readable.

## Basic If Block

```svelte
<script>
  let isLoggedIn = $state(false);
</script>

{#if isLoggedIn}
  <p>Welcome back!</p>
  <button onclick={() => isLoggedIn = false}>Log out</button>
{/if}
```

Content inside the `{#if}` block only appears when the condition is true.

## If-Else

Use `{:else}` to show alternative content:

```svelte
{#if isLoggedIn}
  <p>Welcome back!</p>
  <button onclick={() => isLoggedIn = false}>Log out</button>
{:else}
  <p>Please log in to continue.</p>
  <button onclick={() => isLoggedIn = true}>Log in</button>
{/if}
```

## Multiple Conditions with {:else if}

```svelte
<script>
  let temperature = $state(25);
</script>

{#if temperature > 30}
  <p>🔥 It's hot! Stay hydrated.</p>
{:else if temperature > 20}
  <p>☀️ Nice weather!</p>
{:else if temperature > 10}
  <p>🧥 Bring a jacket.</p>
{:else}
  <p>❄️ It's cold! Bundle up.</p>
{/if}
```

## Any Expression Works

You can use any JavaScript expression as the condition:

```svelte
<script>
  let items = $state(['apple', 'banana']);
  let user = $state({ role: 'admin' });
</script>

{#if items.length > 0}
  <p>You have {items.length} items</p>
{/if}

{#if user.role === 'admin'}
  <button>Delete All</button>
{/if}

{#if items.includes('apple')}
  <p>You have apples!</p>
{/if}
```

## Block Syntax Explained

Svelte's block tags follow a pattern:
- `{#...}` — Opens a block
- `{:...}` — Continues a block (else, else if, then, catch)
- `{/...}` — Closes a block

```svelte
{#if condition}     <!-- Opens -->
  content
{:else}             <!-- Continues -->
  alternative
{/if}               <!-- Closes -->
```

## Resources

- [{#if} Documentation](https://svelte.dev/docs/svelte/if) — Official docs for conditional rendering.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*