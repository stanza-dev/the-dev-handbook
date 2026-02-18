---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-builder-pattern"
---

# Builder Pattern

The Builder pattern is useful for constructing complex objects step-by-step. In TypeScript, we can use it to ensure all required fields are set.

```typescript
class RequestBuilder {
  private url: string = '';
  private method: 'GET' | 'POST' = 'GET';

  setUrl(url: string): this {
    this.url = url;
    return this;
  }

  setMethod(method: 'GET' | 'POST'): this {
    this.method = method;
    return this;
  }

  build() {
    return { url: this.url, method: this.method };
  }
}
```

Using `this` as a return type allows for method chaining in subclasses too.

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*