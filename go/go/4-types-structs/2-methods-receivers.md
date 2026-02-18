---
source_course: "go"
source_lesson: "go-methods-receivers"
---

# Methods

You can define methods on types. The argument between `func` and the method name is called the **receiver**.

```go
// Value receiver (receives a copy)
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// Pointer receiver (can modify the struct)
func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

## When to use Pointer Receivers?
1.  When you need to modify the receiver.
2.  When the struct is large (to avoid copying).
3.  **Consistency:** If one method needs a pointer, use pointer receivers for all methods.

## Method Sets

*   Value `v` can call both value and pointer receiver methods (Go auto-converts).
*   Pointer `*v` can call both as well.
*   But when implementing interfaces, the method set matters!

## Methods on Any Type

You can define methods on any type in your package:

```go
type MyInt int

func (n MyInt) Double() MyInt {
    return n * 2
}
```

## Code Examples

**Struct Methods**

```go
type Rect struct {
    width, height int
}

func (r *Rect) Area() int {
    return r.width * r.height
}

func (r *Rect) Resize(w, h int) {
    r.width = w
    r.height = h
}

func main() {
    r := Rect{width: 10, height: 5}
    fmt.Println("Area:", r.Area())
    r.Resize(20, 10)
    fmt.Println("New Area:", r.Area())
}
```


## Resources

- [A Tour of Go - Methods](https://go.dev/tour/methods/1) — Interactive methods tutorial
- [Effective Go - Methods](https://go.dev/doc/effective_go#methods) — Method design guidelines

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*