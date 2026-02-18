---
source_course: "rust-functional"
source_lesson: "rust-func-family-pattern"
---

# Simulating HKTs with Type Families

The "family pattern" uses associated types to simulate HKTs.

## The Pattern

```rust
// Define a "family" trait
trait TypeFamily {
    type Member<T>;
}

// Implement for specific container types
struct OptionFamily;
impl TypeFamily for OptionFamily {
    type Member<T> = Option<T>;
}

struct VecFamily;
impl TypeFamily for VecFamily {
    type Member<T> = Vec<T>;
}
```

## Using the Family

```rust
trait Functor: TypeFamily {
    fn fmap<A, B, F>(fa: Self::Member<A>, f: F) -> Self::Member<B>
    where
        F: FnMut(A) -> B;
}

impl Functor for OptionFamily {
    fn fmap<A, B, F>(fa: Option<A>, f: F) -> Option<B>
    where
        F: FnMut(A) -> B,
    {
        fa.map(f)
    }
}

impl Functor for VecFamily {
    fn fmap<A, B, F>(fa: Vec<A>, f: F) -> Vec<B>
    where
        F: FnMut(A) -> B,
    {
        fa.into_iter().map(f).collect()
    }
}
```

## Generic Code Over Families

```rust
fn double_all<Fam: Functor>(container: Fam::Member<i32>) -> Fam::Member<i32> {
    Fam::fmap(container, |x| x * 2)
}

let opt = double_all::<OptionFamily>(Some(5));  // Some(10)
let vec = double_all::<VecFamily>(vec![1, 2]);  // [2, 4]
```

## Limitations

- Verbose syntax
- Requires explicit type family parameter
- Less ergonomic than true HKTs
- But it works!

## Code Examples

**Full family pattern example**

```rust
// Complete example with Applicative
trait TypeFamily {
    type Member<T>;
}

trait Functor: TypeFamily {
    fn fmap<A, B>(fa: Self::Member<A>, f: impl FnMut(A) -> B) -> Self::Member<B>;
}

trait Applicative: Functor {
    fn pure<A>(a: A) -> Self::Member<A>;
    fn apply<A, B>(
        ff: Self::Member<impl FnMut(A) -> B>,
        fa: Self::Member<A>
    ) -> Self::Member<B>;
}

// Option implementation
struct OptionFamily;

impl TypeFamily for OptionFamily {
    type Member<T> = Option<T>;
}

impl Functor for OptionFamily {
    fn fmap<A, B>(fa: Option<A>, mut f: impl FnMut(A) -> B) -> Option<B> {
        fa.map(|a| f(a))
    }
}

impl Applicative for OptionFamily {
    fn pure<A>(a: A) -> Option<A> {
        Some(a)
    }
    
    fn apply<A, B>(
        ff: Option<impl FnMut(A) -> B>,
        fa: Option<A>
    ) -> Option<B> {
        match (ff, fa) {
            (Some(mut f), Some(a)) => Some(f(a)),
            _ => None,
        }
    }
}

// Now we can write generic applicative code!
fn lift2<Fam: Applicative, A, B, C>(
    f: impl Fn(A, B) -> C + Clone,
    fa: Fam::Member<A>,
    fb: Fam::Member<B>,
) -> Fam::Member<C>
where
    Fam::Member<A>: Clone,
{
    let ff = Fam::fmap(fa, move |a| {
        let f = f.clone();
        move |b| f(a.clone(), b)
    });
    // This is getting complicated - HKTs would help!
    todo!()
}
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*