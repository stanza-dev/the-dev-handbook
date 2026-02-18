---
source_course: "go"
source_lesson: "go-basic-types"
---

# Numbers, Strings & Booleans

## Introduction

Every program manipulates data, and data comes in types. Go provides a carefully designed set of primitive types optimized for modern computing: integers of various sizes, floating-point numbers for decimals, strings for text, and booleans for logic. Understanding these types—and when to use each—is essential for writing correct, efficient Go code.

## Key Concepts

**Primitive Types**: Built-in types like `int`, `string`, and `bool` that form the building blocks of all data structures.

**Rune**: Go's term for a Unicode code point, aliased to `int32`, representing a single character.

**Byte**: An alias for `uint8`, commonly used for raw binary data.

**Explicit Conversion**: Go never implicitly converts between types; you must explicitly cast using `T(value)` syntax.

## Real World Context

Choosing the right numeric type affects memory usage and performance. An `int64` uses 8 bytes while `int8` uses 1—significant when processing millions of records. Understanding that strings are UTF-8 byte sequences prevents bugs when handling international text. Financial applications often avoid `float64` due to precision issues, preferring integer cents or decimal libraries.

## Deep Dive

### Integer Types

```go
// Signed (can be negative)
var i8  int8   // -128 to 127
var i16 int16  // -32768 to 32767
var i32 int32  // -2^31 to 2^31-1
var i64 int64  // -2^63 to 2^63-1
var i   int    // Platform-dependent (32 or 64 bit)

// Unsigned (positive only)
var u8  uint8  // 0 to 255 (byte)
var u64 uint64 // 0 to 2^64-1
```

### Floating Point

```go
var f32 float32 = 3.14       // 7 digits precision
var f64 float64 = 3.14159265 // 15 digits precision (default)
```

### Strings and Runes

```go
s := "Hello, 世界"
fmt.Println(len(s))                    // 13 (bytes)
fmt.Println(utf8.RuneCountInString(s)) // 9 (runes/characters)

// Iterate by rune
for i, r := range s {
    fmt.Printf("%d: %c\n", i, r)
}

// Rune literal
var r rune = '世'  // int32 value 19990
```

### Type Conversions

```go
var i int = 42
var f float64 = float64(i)  // Explicit conversion required
var s string = strconv.Itoa(i)  // int to string
```

## Common Pitfalls

1. **Assuming `len(s)` returns character count**: For UTF-8 strings with non-ASCII characters, `len()` returns bytes, not runes. Use `utf8.RuneCountInString()`.

2. **Integer overflow silently wraps**: Adding 1 to `int8(127)` gives `-128` with no error. Use larger types or check bounds.

3. **Float comparison with `==`**: Due to precision limits, `0.1 + 0.2 == 0.3` is `false`. Compare with a small epsilon tolerance.

## Best Practices

- Use `int` for general integers unless you need a specific size.
- Use `float64` over `float32` unless memory is critical.
- Use `rune` when processing individual characters.
- Use `byte` (or `[]byte`) for binary data.
- Use `strconv` package for string-to-number conversions.

## Summary

Go provides sized integer types (`int8` to `int64`, `uint8` to `uint64`), floating-point types (`float32`, `float64`), strings (UTF-8 byte sequences), and booleans. The `rune` type represents Unicode code points. Go requires explicit type conversions—there's no implicit casting between numeric types.

## Code Examples

**Basic Types**

```go
package main

import "fmt"

func main() {
    // Integer types
    var i8 int8 = 127
    var u8 uint8 = 255
    
    // Floating point
    var f64 float64 = 3.14159
    
    // Runes (Unicode code points)
    var r rune = '世'
    fmt.Printf("%c = %d\n", r, r) // 世 = 19990
}
```


## Resources

- [A Tour of Go - Basic Types](https://go.dev/tour/basics/11) — Interactive tour of Go's basic types
- [Go Blog - Strings, bytes, runes](https://go.dev/blog/strings) — Deep dive into Go's string implementation

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*