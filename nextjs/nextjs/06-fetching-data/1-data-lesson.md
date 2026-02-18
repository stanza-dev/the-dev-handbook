---
source_course: "nextjs"
source_lesson: "nextjs-fetching-data-lesson"
---

# Fetching Data

In the App Router, you can fetch data directly inside Server Components using `async` and `await`.

## using `fetch`

Next.js extends the native `fetch` API to allow caching and revalidation configuration.

```tsx
async function getData() {
  const res = await fetch('https://api.example.com/data')
  if (!res.ok) throw new Error('Failed to fetch data')
  return res.json()
}

export default async function Page() {
  const data = await getData()
  return <main>{data.title}</main>
}
```

By default, `fetch` requests are cached. You can control this with `cache: 'no-store'` or `next: { revalidate: 3600 }`.

## Code Examples

**Direct fetch in a Server Component**

```tsx
export default async function Page() {
  const data = await fetch('https://api...').then(res => res.json())
  return <div>{data.name}</div>
}
```


---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*