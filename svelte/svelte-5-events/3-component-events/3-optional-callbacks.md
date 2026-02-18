---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-optional-callbacks"
---

# Optional Event Callbacks

Not every parent needs to handle every event. Learn how to make callbacks optional while keeping your components safe.

## The Problem: Undefined Callbacks

```svelte
<!-- Child.svelte -->
<script>
  let { ondelete } = $props();
</script>

<!-- CRASH if parent doesn't pass ondelete! -->
<button onclick={() => ondelete(id)}>Delete</button>
```

If the parent doesn't provide `ondelete`, calling it will throw an error.

## Solution 1: Default No-Op Function

Provide an empty function as default:

```svelte
<script>
  let { ondelete = () => {} } = $props();
</script>

<!-- Safe - calls empty function if not provided -->
<button onclick={() => ondelete(id)}>Delete</button>
```

## Solution 2: Conditional Call

Check if the callback exists before calling:

```svelte
<script>
  let { ondelete } = $props();
</script>

<button onclick={() => ondelete?.(id)}>Delete</button>
```

The `?.()` is optional chaining for functions - it only calls if `ondelete` is defined.

## Solution 3: TypeScript with Optional Props

```svelte
<script lang="ts">
  type Props = {
    id: number;
    ondelete?: (id: number) => void;  // Optional
    onedit?: (id: number) => void;     // Optional
    onsave: (id: number) => void;      // Required
  };
  
  let { id, ondelete, onedit, onsave }: Props = $props();
</script>

<button onclick={() => ondelete?.(id)}>Delete</button>
<button onclick={() => onedit?.(id)}>Edit</button>
<button onclick={() => onsave(id)}>Save</button> <!-- Always exists -->
```

## Combining Both Approaches

For the best DX, combine defaults with TypeScript:

```svelte
<script lang="ts">
  type Props = {
    onchange?: (value: string) => void;
    onsubmit?: (data: FormData) => Promise<void>;
  };
  
  let { 
    onchange = () => {}, 
    onsubmit = async () => {} 
  }: Props = $props();
  
  // Now you can always call them safely
  function handleInput(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    onchange(value); // Safe - has default
  }
</script>
```

## When to Require vs Make Optional

**Required callbacks:**
- Core functionality that doesn't make sense without them
- Actions that MUST trigger parent updates

**Optional callbacks:**
- Nice-to-have hooks (analytics, logging)
- Events that parents might not care about
- Progressive enhancement features

📖 [Props documentation](https://svelte.dev/docs/svelte/$props)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*