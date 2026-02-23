---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-option-pattern"
---

# The Option/Maybe Pattern

## Introduction
`null` and `undefined` are the most common sources of runtime errors in JavaScript. The Option pattern (also called Maybe) wraps a value that might not exist, forcing the caller to handle both cases explicitly. Combined with TypeScript's type system, this eliminates `Cannot read property of undefined` errors.

## Key Concepts
- **Option<T>**: A type that is either `Some<T>` (contains a value) or `None` (empty).
- **Map**: Transforms the inner value without unwrapping — `Some(2).map(x => x * 3)` produces `Some(6)`, while `None.map(...)` stays `None`.
- **FlatMap (chain)**: Like map, but the transformation itself returns an Option, preventing nested `Option<Option<T>>`.
- **GetOrElse**: Unwraps the value with a fallback: `option.getOrElse(defaultValue)`.

## Real World Context
Dictionary lookups, database queries, and configuration reads all return values that may not exist. Instead of checking `if (result !== null)` at every call site, the Option pattern centralizes null-handling and makes it composable.

## Deep Dive
### Defining Option

Using a discriminated union:

```typescript
type Option<T> = { tag: "some"; value: T } | { tag: "none" };

function some<T>(value: T): Option<T> {
    return { tag: "some", value };
}

function none<T>(): Option<T> {
    return { tag: "none" };
}
```

### Map and FlatMap

```typescript
function map<T, U>(opt: Option<T>, fn: (value: T) => U): Option<U> {
    return opt.tag === "some" ? some(fn(opt.value)) : none();
}

function flatMap<T, U>(opt: Option<T>, fn: (value: T) => Option<U>): Option<U> {
    return opt.tag === "some" ? fn(opt.value) : none();
}

function getOrElse<T>(opt: Option<T>, fallback: T): T {
    return opt.tag === "some" ? opt.value : fallback;
}
```

### Practical Example

```typescript
type User = { name: string; addressId?: string };
type Address = { city: string };

const users = new Map<string, User>();
const addresses = new Map<string, Address>();

function findUser(id: string): Option<User> {
    const user = users.get(id);
    return user ? some(user) : none();
}

function findAddress(id: string): Option<Address> {
    const addr = addresses.get(id);
    return addr ? some(addr) : none();
}

function getUserCity(userId: string): string {
    const city = flatMap(
        flatMap(findUser(userId), (user) =>
            user.addressId ? some(user.addressId) : none()
        ),
        (addrId) => map(findAddress(addrId), (addr) => addr.city)
    );
    return getOrElse(city, "Unknown");
}
```

Each step in the chain can fail, but the None propagates automatically — no nested `if` checks required.

## Common Pitfalls
1. **Mixing Option with null checks** — If part of your code uses Option and another part uses raw null, you lose consistency. Pick one approach per module boundary.
2. **Overusing Option for simple cases** — If a function always returns a value, do not wrap it in Option. Use Option only when absence is a real possibility.

## Best Practices
1. **Use Option at boundaries** — Wrap nullable external data (database lookups, API responses) into Option at the entry point, then operate on Options internally.
2. **Consider fp-ts or Effect** — For production code, use a battle-tested library rather than rolling your own Option implementation.

## Summary
- Option<T> encodes the presence or absence of a value in the type system.
- `map` transforms the inner value; `flatMap` chains operations that may also fail.
- `getOrElse` provides a safe unwrap with a fallback.
- Use Option at module boundaries to eliminate null-check sprawl.

## Code Examples

**A type-safe dictionary lookup that returns Option instead of undefined — the caller must handle both cases**

```typescript
type Option<T> = { tag: "some"; value: T } | { tag: "none" };

const some = <T>(value: T): Option<T> => ({ tag: "some", value });
const none = <T>(): Option<T> => ({ tag: "none" });

// Safe dictionary lookup returning Option
function lookup<K, V>(map: Map<K, V>, key: K): Option<V> {
    const value = map.get(key);
    return value !== undefined ? some(value) : none();
}

const settings = new Map([["theme", "dark"], ["lang", "en"]]);

const theme = lookup(settings, "theme");  // { tag: "some", value: "dark" }
const missing = lookup(settings, "font"); // { tag: "none" }
```


## Resources

- [fp-ts Option module](https://gcanti.github.io/fp-ts/modules/Option.ts.html) — Production-ready Option implementation in the fp-ts library

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*