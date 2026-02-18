---
source_course: "rust-functional"
source_lesson: "rust-func-hkt-problem"
---

# Higher-Kinded Types

Rust doesn't have HKTs. This limits some functional patterns.

## The Problem

```rust
// We can't write this:
trait Functor<F<_>> {  // F is a type constructor
    fn fmap<A, B>(self: F<A>, f: impl Fn(A) -> B) -> F<B>;
}

// F<_> would be Option, Vec, Result, etc.
// But Rust doesn't support this syntax
```

## Why It Matters

```rust
// We want to abstract over "container types"
// Option<A> -> Option<B>
// Vec<A> -> Vec<B>
// Result<A, E> -> Result<B, E>

// All have map, but we can't express the pattern generically
```

## Rust's Workaround: Associated Types

```rust
// Instead of F<A>, use associated type families
trait Container {
    type Item;
    type WithItem<T>: Container<Item = T>;
    
    fn map<B, F>(self, f: F) -> Self::WithItem<B>
    where
        F: FnMut(Self::Item) -> B;
}
```

## The Limitation

```rust
// Can't express:
fn twice<F: Functor, A>(x: F<A>) -> F<F<A>> { ... }

// Have to be concrete:
fn twice_option<A>(x: Option<A>) -> Option<Option<A>> {
    Some(x)
}

fn twice_vec<A>(x: Vec<A>) -> Vec<Vec<A>> {
    vec![x]
}
```

See [GATs](https://blog.rust-lang.org/2022/10/28/gats-stabilization.html) for partial solutions.

## Code Examples

**Concrete monad implementations**

```rust
// What we'd like:
// trait Monad<M<_>> {
//     fn pure<A>(a: A) -> M<A>;
//     fn bind<A, B>(ma: M<A>, f: impl Fn(A) -> M<B>) -> M<B>;
// }

// What we have to do instead - be concrete:

trait OptionMonad {
    fn pure<A>(a: A) -> Option<A> {
        Some(a)
    }
    
    fn bind<A, B>(ma: Option<A>, f: impl FnOnce(A) -> Option<B>) -> Option<B> {
        ma.and_then(f)
    }
}

trait VecMonad {
    fn pure<A>(a: A) -> Vec<A> {
        vec![a]
    }
    
    fn bind<A, B>(ma: Vec<A>, f: impl Fn(A) -> Vec<B>) -> Vec<B> {
        ma.into_iter().flat_map(f).collect()
    }
}

// Usage requires knowing the concrete type
fn example() {
    let x = OptionMonad::pure(5);
    let y = OptionMonad::bind(x, |n| Some(n * 2));
}
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*