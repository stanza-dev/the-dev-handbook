---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-each-loops"
---

# Rendering Lists

Most applications need to display lists of items — products, messages, users, etc. The `{#each}` block handles this elegantly.

## Basic Each Loop

```svelte
<script>
  let fruits = $state(['Apple', 'Banana', 'Cherry']);
</script>

<ul>
  {#each fruits as fruit}
    <li>{fruit}</li>
  {/each}
</ul>
```

Output:
- Apple
- Banana  
- Cherry

## Getting the Index

Need the position? Use a second parameter:

```svelte
{#each fruits as fruit, index}
  <li>{index + 1}. {fruit}</li>
{/each}
```

Output:
1. Apple
2. Banana
3. Cherry

## Looping Over Objects

```svelte
<script>
  let users = $state([
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' },
    { id: 3, name: 'Charlie', email: 'charlie@example.com' }
  ]);
</script>

<table>
  {#each users as user}
    <tr>
      <td>{user.name}</td>
      <td>{user.email}</td>
    </tr>
  {/each}
</table>
```

## Destructuring in Each

Make it cleaner with destructuring:

```svelte
{#each users as { name, email }}
  <tr>
    <td>{name}</td>
    <td>{email}</td>
  </tr>
{/each}
```

## Keyed Each Blocks (Important!)

When items can be **added, removed, or reordered**, you MUST provide a unique key:

```svelte
{#each users as user (user.id)}
  <UserCard {user} />
{/each}
```

The key `(user.id)` tells Svelte how to identify each item. Without it:
- Animations may break
- Component state may get mixed up
- Performance suffers

**Rule of thumb**: Always use keys when:
- Items have unique IDs
- Items can be reordered
- Items can be added/removed

## Empty Lists with {:else}

```svelte
{#each items as item}
  <p>{item}</p>
{:else}
  <p>No items yet. Add some!</p>
{/each}
```

The `{:else}` block renders when the array is empty.

## Resources

- [{#each} Documentation](https://svelte.dev/docs/svelte/each) — Complete guide to iterating over arrays in Svelte.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*