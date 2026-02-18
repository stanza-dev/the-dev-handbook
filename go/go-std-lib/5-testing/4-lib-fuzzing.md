---
source_course: "go-std-lib"
source_lesson: "go-std-lib-fuzzing"
---

# Fuzz Testing (Go 1.18+)

Fuzzing automatically generates test inputs to find edge cases and bugs.

## Writing a Fuzz Test

```go
func FuzzParse(f *testing.F) {
    // Add seed corpus
    f.Add("hello")
    f.Add("world")
    f.Add("")
    
    f.Fuzz(func(t *testing.T, input string) {
        // This runs with random inputs
        result, err := Parse(input)
        if err != nil {
            return  // Expected for invalid input
        }
        
        // Round-trip check
        encoded := Encode(result)
        if encoded != input {
            t.Errorf("round-trip failed: %q -> %v -> %q", input, result, encoded)
        }
    })
}
```

## Running Fuzz Tests

```bash
go test -fuzz=FuzzParse           # Run fuzzer
go test -fuzz=FuzzParse -fuzztime=30s  # Run for 30 seconds
go test -run=FuzzParse            # Run with corpus only (no fuzzing)
```

## Corpus

Failed inputs are saved to `testdata/fuzz/FuzzName/` for regression testing.

## Code Examples

**Fuzz Test Example**

```go
func FuzzReverse(f *testing.F) {
    f.Add("hello")
    f.Add("世界")
    
    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        doubleRev := Reverse(rev)
        
        if s != doubleRev {
            t.Errorf("Reverse(Reverse(%q)) = %q", s, doubleRev)
        }
    })
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*