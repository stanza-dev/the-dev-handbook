---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-checkboxes-selects"
---

# Beyond Text Inputs

Svelte provides specialized bindings for different form controls.

## Single Checkbox: bind:checked

For checkboxes, bind to the `checked` property:

```svelte
<script>
  let accepted = $state(false);
  let newsletter = $state(true);
</script>

<label>
  <input type="checkbox" bind:checked={accepted} />
  I accept the terms and conditions
</label>

<label>
  <input type="checkbox" bind:checked={newsletter} />
  Subscribe to newsletter
</label>

<button disabled={!accepted}>Continue</button>
```

## Checkbox Groups: bind:group

For multiple checkboxes that add to an array:

```svelte
<script>
  let selectedFruits = $state([]);
</script>

<label>
  <input type="checkbox" bind:group={selectedFruits} value="apple" />
  Apple
</label>
<label>
  <input type="checkbox" bind:group={selectedFruits} value="banana" />
  Banana
</label>
<label>
  <input type="checkbox" bind:group={selectedFruits} value="cherry" />
  Cherry
</label>

<p>Selected: {selectedFruits.join(', ') || 'None'}</p>
```

## Radio Buttons: bind:group

Radio buttons work similarly, but the variable holds a single value:

```svelte
<script>
  let theme = $state('light');
</script>

<label>
  <input type="radio" bind:group={theme} value="light" />
  Light Mode
</label>
<label>
  <input type="radio" bind:group={theme} value="dark" />
  Dark Mode
</label>
<label>
  <input type="radio" bind:group={theme} value="system" />
  System Default
</label>

<p>Current theme: {theme}</p>
```

## Select Dropdowns

```svelte
<script>
  let selectedCountry = $state('');
  
  const countries = [
    { code: 'us', name: 'United States' },
    { code: 'uk', name: 'United Kingdom' },
    { code: 'fr', name: 'France' }
  ];
</script>

<select bind:value={selectedCountry}>
  <option value="">Select a country...</option>
  {#each countries as country}
    <option value={country.code}>{country.name}</option>
  {/each}
</select>

<p>Selected: {selectedCountry || 'Nothing'}</p>
```

## Multi-Select

For `<select multiple>`, bind to an array:

```svelte
<script>
  let selectedColors = $state([]);
</script>

<select multiple bind:value={selectedColors}>
  <option value="red">Red</option>
  <option value="green">Green</option>
  <option value="blue">Blue</option>
</select>
```

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*