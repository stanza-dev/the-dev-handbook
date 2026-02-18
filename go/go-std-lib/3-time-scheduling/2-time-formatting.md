---
source_course: "go-std-lib"
source_lesson: "go-std-lib-time-formatting"
---

# Go's Unique Time Format

Go uses a unique reference time for formatting: `Mon Jan 2 15:04:05 MST 2006`.

The mnemonic: 1 2 3 4 5 6 7 (Month=1, Day=2, Hour=15=3PM, Min=4, Sec=5, Year=6, Timezone=7)

## Common Layouts

```go
time.RFC3339      // 2006-01-02T15:04:05Z07:00
time.RFC1123      // Mon, 02 Jan 2006 15:04:05 MST
time.Kitchen      // 3:04PM
"2006-01-02"      // YYYY-MM-DD
"15:04:05"        // HH:MM:SS
"Jan 2, 2006"     // Jan 2, 2006
```

## Formatting

```go
t := time.Now()
fmt.Println(t.Format("2006-01-02"))        // 2024-01-15
fmt.Println(t.Format(time.RFC3339))        // 2024-01-15T14:30:00Z
fmt.Println(t.Format("Monday, January 2")) // Monday, January 15
```

## Parsing

```go
t, err := time.Parse("2006-01-02", "2024-01-15")
t, err := time.Parse(time.RFC3339, "2024-01-15T14:30:00Z")

// With location
loc, _ := time.LoadLocation("America/New_York")
t, err := time.ParseInLocation("2006-01-02 15:04", "2024-01-15 14:30", loc)
```

## Code Examples

**Time Formatting**

```go
// Common formats
t := time.Now()
fmt.Println(t.Format("2006-01-02 15:04:05")) // ISO-like
fmt.Println(t.Format("Jan 02, 2006"))         // Human readable
fmt.Println(t.Format(time.RFC3339))           // Standard
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*