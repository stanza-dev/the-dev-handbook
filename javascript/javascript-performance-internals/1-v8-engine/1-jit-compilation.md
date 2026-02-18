---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-jit-compilation"
---

# JIT Compilation & Optimization

## Introduction

Understanding how JavaScript engines work helps you write faster code. Modern engines like V8 use sophisticated Just-In-Time compilation to achieve near-native performance.

## Key Concepts

**JIT Compilation**: Compiling code at runtime, not ahead of time.

**Hot Path**: Code that runs frequently and gets optimized.

**Deoptimization**: Falling back from optimized to unoptimized code.

## Deep Dive

### V8 Pipeline

```
Source Code
    ↓
Parser → AST (Abstract Syntax Tree)
    ↓
Ignition (Interpreter) → Bytecode
    ↓ (hot paths)
TurboFan (Compiler) → Optimized Machine Code
```

1. **Parser**: Generates AST from source code.
2. **Ignition**: Interprets bytecode, collects profiling data.
3. **TurboFan**: Compiles hot functions to optimized machine code.

### Tiered Compilation

```javascript
// Cold code - runs in interpreter
function rarelyUsed() { ... }

// Hot code - gets compiled to machine code
for (let i = 0; i < 100000; i++) {
  frequentlyUsed();  // TurboFan kicks in
}
```

### Deoptimization

```javascript
function add(a, b) {
  return a + b;
}

// Optimized for numbers
add(1, 2);
add(3, 4);
// ...
add(5, 6);  // TurboFan optimizes for integers

// Deoptimization!
add('hello', 'world');  // Different types → back to interpreter
```

## Common Pitfalls

1. **Megamorphic call sites**: Calling functions with many different object shapes.
2. **Hidden class transitions**: Adding properties inconsistently.
3. **Bail-out patterns**: try/catch in hot loops (older engines).

## Best Practices

- **Keep types consistent**: Don't mix types in hot paths.
- **Avoid deleting properties**: Changes hidden class.
- **Initialize all properties in constructors**: Consistent shape.
- **Profile before optimizing**: Don't guess.

## Summary

V8 uses Ignition (interpreter) and TurboFan (optimizing compiler). Hot code gets compiled to machine code. Type consistency helps optimization. Deoptimization happens when assumptions break.

## Resources

- [V8 Blog](https://v8.dev/blog) — V8 engine blog

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*