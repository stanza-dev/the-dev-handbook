---
source_course: "javascript-performance-internals"
source_lesson: "javascript-performance-internals-jit-compilation"
---

## Introduction

JavaScript is often called an "interpreted" language, but modern engines like V8 go far beyond simple interpretation. They use **Just-In-Time (JIT) compilation** to translate your JavaScript into highly optimized machine code at runtime. Understanding how JIT compilation works is the foundation for writing performant JavaScript.

When you run JavaScript in Chrome or Node.js, V8 doesn't just read your code line by line. It employs a sophisticated multi-tier compilation pipeline that balances startup speed with peak performance. Code that runs once is interpreted quickly; code that runs thousands of times gets compiled into blazing-fast machine code.

## Key Concepts

- **JIT Compilation**: A technique where code is compiled to machine code at runtime rather than ahead of time. V8 compiles JavaScript functions as they execute, using runtime information to generate faster code than a static compiler could.

- **Ignition Interpreter**: V8's bytecode interpreter. When JavaScript is first executed, Ignition compiles it to a compact bytecode format and interprets it. This provides fast startup while collecting type feedback for later optimization.

- **TurboFan Compiler**: V8's optimizing compiler. When Ignition identifies a function as "hot" (called frequently), TurboFan takes the bytecode and type feedback to produce highly optimized machine code with speculative optimizations.

- **Hot Functions**: Functions that are called repeatedly and identified by the engine as optimization candidates. V8 tracks call counts and loop iterations to decide when to trigger TurboFan compilation.

## Real World Context

JIT compilation is why modern JavaScript can rival C++ performance for compute-heavy tasks. Libraries like TensorFlow.js, image processing tools, and real-time data visualization frameworks all depend on V8's ability to optimize hot loops and frequently-called functions. When a React component's render function runs thousands of times, JIT ensures subsequent calls are nearly as fast as native code.

Understanding JIT also explains why "warming up" a server matters in production. The first few hundred requests may be slower as V8 interprets code and gathers type feedback. Once TurboFan kicks in, throughput can increase dramatically.

## Deep Dive

V8's compilation pipeline works in stages:

1. **Parsing**: Source code is parsed into an Abstract Syntax Tree (AST).
2. **Bytecode Generation**: Ignition walks the AST and generates bytecode.
3. **Interpretation**: Ignition executes bytecode, collecting type feedback at each operation.
4. **Optimization**: When a function becomes hot, TurboFan uses type feedback to compile optimized machine code.
5. **Deoptimization**: If runtime types violate TurboFan's assumptions, the engine "deoptimizes" back to bytecode.

Here is an example of code that V8 can optimize well:

```javascript
// Monomorphic function - always receives numbers
function multiply(a, b) {
  return a * b;
}

for (let i = 0; i < 10000; i++) {
  multiply(i, i + 1); // TurboFan sees: always numbers
}
```

After several thousand calls, TurboFan compiles `multiply` with the assumption that `a` and `b` are always numbers. It emits a single machine instruction for the multiplication, skipping all type checks.

Now consider this pattern that defeats optimization:

```javascript
function add(a, b) {
  return a + b;
}

add(1, 2);           // number + number
add('foo', 'bar');   // string + string
add(true, 1);        // boolean + number
```

The `+` operator behaves differently for numbers, strings, and mixed types. When V8 sees multiple types flowing into the same function, it cannot specialize the machine code and must generate slower, generic code paths.

## Common Pitfalls

- **Mixing types in hot functions**: Passing different types to the same function (numbers, then strings) forces V8 to generate generic code or deoptimize. Keep function call sites monomorphic for best performance.

- **Assuming all code is optimized**: Only hot functions get compiled by TurboFan. Code that runs once stays as interpreted bytecode, which is fine because interpretation is fast for cold code.

- **Premature optimization by rewriting for the engine**: V8's heuristics change between versions. Write clear, idiomatic JavaScript first, then profile to find actual bottlenecks.

## Best Practices

- **Keep functions monomorphic**: Call each function with consistent argument types. If a function processes both numbers and strings, consider splitting it into two specialized functions.

- **Let hot loops be simple**: V8 optimizes tight loops aggressively. Avoid complex control flow, try-catch blocks, or `eval` inside performance-critical loops.

- **Profile before optimizing**: Use Chrome DevTools Performance tab or Node.js `--trace-opt` and `--trace-deopt` flags to see what V8 actually optimizes and deoptimizes.

## Summary

V8's JIT compilation pipeline transforms JavaScript from interpreted bytecode into optimized machine code at runtime. Ignition provides fast startup by interpreting bytecode while collecting type feedback. TurboFan uses that feedback to compile hot functions into specialized machine code. Writing monomorphic, type-stable code helps TurboFan generate the fastest possible output.

## Code Examples

**Monomorphic vs megamorphic function calls affecting JIT optimization**

```javascript
// Monomorphic function - optimized by TurboFan
function add(a, b) {
  return a + b;
}

// Always called with numbers → stays optimized
for (let i = 0; i < 10000; i++) {
  add(i, i + 1);
}

// Megamorphic call - causes deoptimization
add('hello', ' world'); // different types!
```


## Resources

- [V8 Blog: TurboFan JIT](https://v8.dev/blog/turbofan-jit) — V8's optimizing compiler architecture

---

> 📘 *This lesson is part of the [JavaScript Under the Hood](https://stanza.dev/courses/javascript-performance-internals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*