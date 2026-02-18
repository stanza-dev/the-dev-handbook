---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-compound-components"
---

# Building Component Families

Compound components are groups of components designed to work together. Snippets enable this pattern elegantly.

## Traditional Approach (Multiple Components)

```svelte
<!-- Old way: separate components for each part -->
<Tabs>
  <TabList>
    <Tab>One</Tab>
    <Tab>Two</Tab>
  </TabList>
  <TabPanels>
    <TabPanel>Content 1</TabPanel>
    <TabPanel>Content 2</TabPanel>
  </TabPanels>
</Tabs>
```

## Snippet Approach (Single Component)

```svelte
<!-- Tabs.svelte -->
<script>
  let { tabs } = $props();
  let activeIndex = $state(0);
</script>

<div class="tabs">
  <div class="tab-list" role="tablist">
    {#each tabs as tab, index}
      <button
        class="tab"
        class:active={index === activeIndex}
        onclick={() => activeIndex = index}
        role="tab"
        aria-selected={index === activeIndex}
      >
        {@render tab.label()}
      </button>
    {/each}
  </div>
  
  <div class="tab-panel" role="tabpanel">
    {@render tabs[activeIndex].content()}
  </div>
</div>
```

```svelte
<!-- Usage -->
<Tabs tabs={[
  {
    label: () => <span>🏠 Home</span>,
    content: () => <div>Home content</div>
  },
  {
    label: () => <span>⚙️ Settings</span>,
    content: () => <SettingsPanel />
  }
]} />
```

Wait — that doesn't work because we can't define snippets inline like that. Let's fix it:

## The Correct Pattern

```svelte
<!-- Usage -->
<Tabs>
  {#snippet tabs()}
    {#snippet tab1Label()}🏠 Home{/snippet}
    {#snippet tab1Content()}<div>Home content</div>{/snippet}
    
    {#snippet tab2Label()}⚙️ Settings{/snippet}
    {#snippet tab2Content()}<SettingsPanel />{/snippet}
  {/snippet}
</Tabs>
```

Actually, let's use a more practical pattern:

## Clean API with Named Snippets

```svelte
<!-- Card.svelte -->
<script>
  let { 
    header,
    children,
    footer,
    image,
    actions
  } = $props();
</script>

<div class="card">
  {#if image}
    <div class="card-image">
      {@render image()}
    </div>
  {/if}
  
  {#if header}
    <header class="card-header">
      {@render header()}
    </header>
  {/if}
  
  <div class="card-body">
    {@render children?.()}
  </div>
  
  {#if actions}
    <div class="card-actions">
      {@render actions()}
    </div>
  {/if}
  
  {#if footer}
    <footer class="card-footer">
      {@render footer()}
    </footer>
  {/if}
</div>
```

```svelte
<!-- Clean usage -->
<Card>
  {#snippet image()}
    <img src={product.image} alt={product.name} />
  {/snippet}
  
  {#snippet header()}
    <h3>{product.name}</h3>
    <span class="price">${product.price}</span>
  {/snippet}
  
  <p>{product.description}</p>
  
  {#snippet actions()}
    <button onclick={() => addToCart(product)}>Add to Cart</button>
    <button onclick={() => addToWishlist(product)}>♡</button>
  {/snippet}
</Card>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*