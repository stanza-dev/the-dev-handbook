---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-template-expressions"
---

# Bringing Data to Life in Your Templates

Svelte's template syntax lets you embed JavaScript expressions directly in your HTML. This is how you create dynamic, interactive user interfaces.

## Curly Brace Expressions

Use single curly braces `{}` to insert JavaScript values into your markup:

```svelte
<script>
  let name = 'Svelte';
  let version = 5;
  let features = ['Runes', 'Snippets', 'Better Performance'];
</script>

<h1>Welcome to {name} {version}!</h1>
<p>Features count: {features.length}</p>
```

## Any JavaScript Expression Works

You're not limited to simple variables. Any valid JavaScript expression can go inside `{}`:

```svelte
<script>
  let price = 99.99;
  let quantity = 3;
  let firstName = 'john';
</script>

<!-- Math -->
<p>Total: ${price * quantity}</p>

<!-- String methods -->
<p>Name: {firstName.toUpperCase()}</p>

<!-- Ternary operator -->
<p>Status: {quantity > 0 ? 'In Stock' : 'Out of Stock'}</p>

<!-- Template literals -->
<p>{`You ordered ${quantity} items`}</p>
```

## HTML Attributes

Use curly braces for dynamic attribute values too:

```svelte
<script>
  let imageSrc = '/photo.jpg';
  let isDisabled = true;
  let buttonClass = 'primary';
</script>

<!-- Dynamic src -->
<img src={imageSrc} alt="A photo" />

<!-- Boolean attributes -->
<button disabled={isDisabled}>Can't click me</button>

<!-- Dynamic classes -->
<button class={buttonClass}>Click me</button>
```

## Shorthand Syntax

When the attribute name matches the variable name, use the shorthand:

```svelte
<script>
  let src = '/photo.jpg';
  let disabled = true;
</script>

<!-- These are equivalent -->
<img src={src} />
<img {src} />       <!-- Shorthand! -->

<button disabled={disabled}>Click</button>
<button {disabled}>Click</button>  <!-- Shorthand! -->
```

## Rendering HTML

By default, expressions are escaped (safe from XSS attacks). To render actual HTML, use `{@html}`:

```svelte
<script>
  let content = '<strong>Bold text</strong>';
</script>

<!-- This shows the literal string -->
<p>{content}</p>  
<!-- Output: <strong>Bold text</strong> -->

<!-- This renders as HTML -->
<p>{@html content}</p>  
<!-- Output: Bold text (actually bold) -->
```

**Warning**: Only use `{@html}` with content you trust! User-generated content could contain malicious scripts.

## Resources

- [Basic Markup](https://svelte.dev/docs/svelte/basic-markup) — Learn about Svelte's template syntax and expressions.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*