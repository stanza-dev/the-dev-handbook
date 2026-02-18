---
source_course: "go-std-lib"
source_lesson: "go-std-lib-pull-iterators"
---

# Push vs Pull

The standard iterator (`func(yield)`) is a **Push** iterator: it drives the loop. Sometimes you want to control the iteration yourself (e.g., zipping two sequences).

## iter.Pull
`iter.Pull` converts a push iterator into a `next` function and a `stop` function.

```go
next, stop := iter.Pull(mySeq)
defer stop()  // Always call stop!

for {
    v, ok := next()
    if !ok {
        break
    }
    fmt.Println(v)
}
```

## Use Cases

*   Zipping multiple iterators together.
*   Manual iteration control.
*   Interleaving from multiple sources.

## Resource Management

**Important:** Always call `stop()` even if you consumed all items. It cleans up the internal goroutine.

## Code Examples

**Zipping two sequences**

```go
func Zip(seq1, seq2 iter.Seq[int]) iter.Seq2[int, int] {
    return func(yield func(int, int) bool) {
        next1, stop1 := iter.Pull(seq1)
        defer stop1()
        next2, stop2 := iter.Pull(seq2)
        defer stop2()

        for {
            v1, ok1 := next1()
            v2, ok2 := next2()
            if !ok1 || !ok2 {
                return
            }
            if !yield(v1, v2) {
                return
            }
        }
    }
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*