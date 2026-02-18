---
source_course: "react-server-components"
source_lesson: "react-server-components-what-are-rsc"
---

# React Server Components: A New Paradigm

React Server Components (RSC) represent a fundamental shift in how we build React applications. They allow components to run **exclusively on the server**, never shipping their code to the browser.

## The Problem RSC Solves

Traditional React apps have limitations:

1. **Large bundles** - All component code ships to the browser
2. **Waterfalls** - Client fetches data after JavaScript loads
3. **Security concerns** - Sensitive logic must be in separate API routes
4. **Backend access** - Can't directly access databases or filesystems

## Server Components vs Client Components

| Aspect | Server Components | Client Components |
|--------|-------------------|-------------------|
| Where they run | Server only | Server (SSR) + Client |
| Bundle size impact | Zero | Adds to bundle |
| Can use hooks | ❌ No | ✅ Yes |
| Can use browser APIs | ❌ No | ✅ Yes |
| Can access backend | ✅ Yes | ❌ No (needs API) |
| Can be async | ✅ Yes | ❌ No |

## The Default: Server Components

In an RSC environment (like Next.js App Router), **all components are Server Components by default**:

```tsx
// This is a Server Component by default
async function ProductPage({ id }) {
  // Direct database access - no API needed!
  const product = await db.products.findUnique({ where: { id } });
  
  return (
    <article>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
    </article>
  );
}
```

## Benefits of Server Components

### 1. Zero Bundle Size

```tsx
// This entire component and its dependencies stay on the server
import { marked } from 'marked';  // 35KB library
import hljs from 'highlight.js'; // 180KB library

async function BlogPost({ slug }) {
  const post = await getPost(slug);
  const html = marked(post.content, {
    highlight: (code) => hljs.highlightAuto(code).value
  });
  
  return <article dangerouslySetInnerHTML={{ __html: html }} />;
}
// Client receives: Just the rendered HTML, not the libraries!
```

### 2. Direct Backend Access

```tsx
async function UserDashboard({ userId }) {
  // Direct database queries
  const user = await prisma.user.findUnique({ where: { id: userId } });
  
  // Access environment variables safely
  const apiKey = process.env.INTERNAL_API_KEY;
  
  // Read from filesystem
  const config = await fs.readFile('./config.json', 'utf-8');
  
  return <Dashboard user={user} />;
}
```

### 3. Automatic Code Splitting

The RSC architecture automatically splits your code - Server Component code never reaches the client.

## Resources

- [Server Components](https://react.dev/reference/rsc/server-components) — Official React documentation for Server Components

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*