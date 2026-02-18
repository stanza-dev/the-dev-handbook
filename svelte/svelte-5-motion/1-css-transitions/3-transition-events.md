---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-transition-events"
---

# Reacting to Transition Lifecycle

Svelte fires events at key moments during transitions, letting you coordinate complex animations.

## Available Events

```svelte
<script>
  import { fly } from 'svelte/transition';
  
  let status = $state('idle');
</script>

{#if visible}
  <div 
    transition:fly={{ y: 100 }}
    onintrostart={() => status = 'entering'}
    onintroend={() => status = 'entered'}
    onoutrostart={() => status = 'leaving'}
    onoutroend={() => status = 'left'}
  >
    Content
  </div>
{/if}

<p>Status: {status}</p>
```

## Event Timeline

```
Element enters DOM:
  ├── introstart fires
  ├── ... animation plays ...
  └── introend fires

Element leaves DOM:
  ├── outrostart fires
  ├── ... animation plays ...
  ├── outroend fires
  └── Element removed from DOM
```

## Practical Example: Loading Button

```svelte
<script>
  import { fade } from 'svelte/transition';
  
  let isLoading = $state(false);
  let buttonText = $state('Submit');
  
  async function handleSubmit() {
    isLoading = true;
    await submitForm();
    isLoading = false;
  }
</script>

<button onclick={handleSubmit} disabled={isLoading}>
  {#if isLoading}
    <span 
      transition:fade={{ duration: 150 }}
      onintroend={() => buttonText = 'Loading...'}
      onoutrostart={() => buttonText = 'Submit'}
    >
      <Spinner />
    </span>
  {/if}
  {buttonText}
</button>
```

## Coordinating Multiple Elements

```svelte
<script>
  let step = $state(1);
  let canProceed = $state(true);
</script>

{#if step === 1}
  <div 
    transition:fly={{ x: -100 }}
    onoutroend={() => canProceed = true}
  >
    Step 1 content
  </div>
{:else}
  <div 
    transition:fly={{ x: 100 }}
    onintrostart={() => canProceed = false}
  >
    Step 2 content
  </div>
{/if}

<button onclick={() => step++} disabled={!canProceed}>
  Next
</button>
```

📖 [Transition events](https://svelte.dev/docs/svelte/transition#Transition-events)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*