---
source_course: "react-accessibility"
source_lesson: "react-accessibility-semantic-elements"
---

# Semantic HTML Elements

Semantic elements convey meaning to browsers and assistive technologies.

## Document Structure

```jsx
function App() {
  return (
    <>
      <header>
        <nav aria-label="Main navigation">
          {/* Navigation links */}
        </nav>
      </header>
      
      <main>
        <article>
          <h1>Article Title</h1>
          <section>
            <h2>Section Heading</h2>
            <p>Content...</p>
          </section>
        </article>
        
        <aside aria-label="Related content">
          {/* Sidebar content */}
        </aside>
      </main>
      
      <footer>
        {/* Footer content */}
      </footer>
    </>
  );
}
```

## Landmark Roles

| Element | Role | Purpose |
|---------|------|----------|
| `<header>` | banner | Site header |
| `<nav>` | navigation | Navigation links |
| `<main>` | main | Primary content |
| `<aside>` | complementary | Supporting content |
| `<footer>` | contentinfo | Site footer |
| `<section>` | region | Thematic grouping |
| `<article>` | article | Self-contained content |

## Headings

Create a logical hierarchy:

```jsx
// ✅ Good: Logical heading order
<main>
  <h1>Page Title</h1>      {/* Only one h1 per page */}
  <section>
    <h2>Section One</h2>
    <h3>Subsection</h3>
  </section>
  <section>
    <h2>Section Two</h2>
  </section>
</main>

// ❌ Bad: Skipping levels for styling
<main>
  <h1>Title</h1>
  <h4>This looks smaller</h4>  {/* Skipped h2, h3! */}
</main>

// ✅ Better: Use CSS for styling
<main>
  <h1>Title</h1>
  <h2 className="text-sm">This looks smaller</h2>
</main>
```

## Lists

```jsx
// Navigation - use list
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

// Definition list for key-value pairs
<dl>
  <dt>Name</dt>
  <dd>John Doe</dd>
  <dt>Email</dt>
  <dd>john@example.com</dd>
</dl>
```

## Tables

```jsx
<table>
  <caption>Monthly Sales Report</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">January</th>
      <td>$10,000</td>
    </tr>
  </tbody>
</table>
```

## Buttons vs Links

```jsx
// Button: Performs an action
<button onClick={handleSave}>Save</button>

// Link: Navigates somewhere
<a href="/settings">Settings</a>

// ❌ Don't use links as buttons
<a href="#" onClick={handleSave}>Save</a>

// ❌ Don't use buttons for navigation
<button onClick={() => navigate('/settings')}>Settings</button>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*