---
source_course: "rust-performance"
source_lesson: "rust-perf-enum-dispatch"
---

# Enum Dispatch vs Dynamic Dispatch

## Introduction

When you need runtime polymorphism in Rust, you have two main options: trait objects (`dyn Trait`) and enum dispatch. Each has different performance characteristics, and choosing correctly can mean a 2-10x difference in hot paths.

## Key Concepts

### Dynamic Dispatch (`dyn Trait`)

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Circle { radius: f64 }
struct Rect { w: f64, h: f64 }

impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
}
impl Shape for Rect {
    fn area(&self) -> f64 { self.w * self.h }
}

fn total_area(shapes: &[Box<dyn Shape>]) -> f64 {
    shapes.iter().map(|s| s.area()).sum()
}
```

Each `.area()` call goes through a vtable pointer — an indirect function call.

### Enum Dispatch

```rust
enum ShapeEnum {
    Circle(Circle),
    Rect(Rect),
}

impl ShapeEnum {
    fn area(&self) -> f64 {
        match self {
            ShapeEnum::Circle(c) => std::f64::consts::PI * c.radius * c.radius,
            ShapeEnum::Rect(r) => r.w * r.h,
        }
    }
}

fn total_area_enum(shapes: &[ShapeEnum]) -> f64 {
    shapes.iter().map(|s| s.area()).sum()
}
```

The `match` compiles to a direct jump table — no indirection.

## Real World Context

ECS (Entity Component System) game engines, parsers, and state machines use enum dispatch for hot-path polymorphism. Libraries like `enum_dispatch` automate the pattern.

## Deep Dive

### Why Enum Dispatch Is Faster

1. **No indirection**: Match compiles to a jump table or branch, not a pointer dereference.
2. **Inlining**: The compiler can inline each match arm. Vtable calls cannot be inlined.
3. **Data locality**: Enum variants are stored inline; `Box<dyn Trait>` scatters data across the heap.
4. **Branch prediction**: CPUs predict match branches well; vtable calls are harder to predict.

### The `enum_dispatch` Crate

```rust
use enum_dispatch::enum_dispatch;

#[enum_dispatch]
trait Shape {
    fn area(&self) -> f64;
}

#[enum_dispatch(Shape)]
enum ShapeEnum {
    Circle,
    Rect,
}
```

This automatically generates the match-based dispatch.

### When to Use Each

| Criterion | Enum Dispatch | dyn Trait |
|-----------|--------------|----------|
| Known set of types | Best | |
| Open set (plugins) | | Best |
| Hot path (millions of calls) | Best | |
| Cold path | Either | Either |
| Binary size concerns | | Better |
| Object safety needed | | Required |

### Benchmark Comparison

Typical benchmark results:

```
enum dispatch:  2.1 ns/call
dyn Trait:     8.4 ns/call
static (impl): 1.8 ns/call
```

Enum dispatch approaches static dispatch speed, while dyn Trait is 3-4x slower due to indirection and failed inlining.

## Common Pitfalls

- Using `Box<dyn Trait>` in a hot loop when all types are known at compile time.
- Creating huge enums with many large variants — Box the large ones.
- Forgetting that enum dispatch requires updating the enum when adding new types.

## Best Practices

- Use enum dispatch when the set of types is known and fixed.
- Use `dyn Trait` when the set of types is open (plugins, user-defined types).
- Benchmark before choosing — the difference matters only in hot paths.
- Consider `enum_dispatch` crate to reduce boilerplate.

## Summary

Enum dispatch eliminates vtable indirection by using match-based jump tables, typically 3-4x faster than `dyn Trait` in hot paths. Use it when the set of types is closed and known. Use `dyn Trait` for open type sets and when binary size matters more than call speed.

## Code Examples

**Enum dispatch vs dynamic dispatch for expression evaluation**

```rust
// Enum dispatch: fast, closed set of types
enum Expr {
    Literal(f64),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
}

impl Expr {
    fn eval(&self) -> f64 {
        match self {
            Expr::Literal(v) => *v,
            Expr::Add(a, b) => a.eval() + b.eval(),
            Expr::Mul(a, b) => a.eval() * b.eval(),
        }
    }
}

// Dynamic dispatch: flexible, open set of types
trait ExprTrait {
    fn eval(&self) -> f64;
}

// Any crate can implement ExprTrait for new types
// But each eval() call goes through a vtable
```


## Resources

- [enum_dispatch crate](https://docs.rs/enum_dispatch/latest/enum_dispatch/) — Crate that automates enum-based dispatch for traits
- [Rust Performance Pitfalls — Dynamic Dispatch](https://nnethercote.github.io/perf-book/type-sizes.html) — Performance implications of dynamic vs static dispatch

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*