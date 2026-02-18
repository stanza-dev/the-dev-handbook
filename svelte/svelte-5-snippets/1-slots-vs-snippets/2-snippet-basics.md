---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-snippet-basics"
---

# Snippet Fundamentals

Let's learn the mechanics of creating and using snippets.

## Defining a Snippet

```svelte
{#snippet button(text, variant)}
  <button class="btn btn-{variant}">
    {text}
  </button>
{/snippet}
```

A snippet is defined with:
- `{#snippet name(parameters)}` — Opening tag with name and optional parameters
- Template content inside
- `{/snippet}` — Closing tag

## Rendering a Snippet

```svelte
{@render button('Click me', 'primary')}
{@render button('Cancel', 'secondary')}
{@render button('Delete', 'danger')}
```

## No Parameters

Snippets don't need parameters:

```svelte
{#snippet logo()}
  <img src="/logo.svg" alt="Company Logo" />
{/snippet}

<header>
  {@render logo()}
</header>
```

**Note**: Even without parameters, use empty parentheses `()` in both definition and render.

## Using Component State

Snippets can access component state:

```svelte
<script>
  let count = $state(0);
</script>

{#snippet counter()}
  <div>
    <button onclick={() => count--}>-</button>
    <span>{count}</span>
    <button onclick={() => count++}>+</button>
  </div>
{/snippet}

{@render counter()}
```

## Conditional Snippets

```svelte
{#snippet status(isOnline)}
  {#if isOnline}
    <span class="badge green">Online</span>
  {:else}
    <span class="badge gray">Offline</span>
  {/if}
{/snippet}

{@render status(user.isOnline)}
```

## Snippets with Complex Markup

```svelte
{#snippet userCard(user)}
  <div class="card">
    <img src={user.avatar} alt="{user.name}'s avatar" />
    <div class="info">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <span class="role">{user.role}</span>
    </div>
    <div class="actions">
      <button onclick={() => editUser(user.id)}>Edit</button>
      <button onclick={() => deleteUser(user.id)}>Delete</button>
    </div>
  </div>
{/snippet}

{#each users as user}
  {@render userCard(user)}
{/each}
```

## Optional Rendering

Check if a snippet exists before rendering:

```svelte
<script>
  let { header } = $props();
</script>

{#if header}
  {@render header()}
{/if}

<!-- Or using optional chaining -->
{@render header?.()}  
```

## Resources

- [{@render} Documentation](https://svelte.dev/docs/svelte/@render) — How to render snippets in Svelte 5.

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*