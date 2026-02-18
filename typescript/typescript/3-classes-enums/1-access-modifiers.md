---
source_course: "typescript"
source_lesson: "typescript-classes-access-modifiers"
---

# Classes

TypeScript adds type annotations and access modifiers to JavaScript classes.

## Access Modifiers

- `public` (default): Accessible anywhere.
- `private`: Accessible only within the class.
- `protected`: Accessible within the class and subclasses.

```typescript
class Person {
  protected name: string;
  private secrets: string[];

  constructor(name: string) {
    this.name = name;
    this.secrets = [];
  }
}

class Employee extends Person {
  sayHello() {
    // Can access protected 'name'
    console.log(`Hi, I'm ${this.name}`); 
    // Cannot access private 'secrets'
    // console.log(this.secrets); // Error
  }
}
```

### Resources
- [TypeScript Handbook: Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*