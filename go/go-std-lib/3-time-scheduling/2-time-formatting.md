---
source_course: "go-std-lib"
source_lesson: "go-std-lib-time-formatting"
---

# Time Formatting & Parsing

## Introduction
Most languages use `YYYY-MM-DD` placeholders for formatting dates. Go takes a completely different approach: it uses a specific reference time, `Mon Jan 2 15:04:05 MST 2006`, as the layout template. Once you learn the mnemonic, formatting and parsing become intuitive and less error-prone than juggling cryptic format codes.

## Key Concepts
- **Reference time**: The fixed date `Mon Jan 2 15:04:05 MST 2006` whose components (month=1, day=2, hour=15, min=04, sec=05, year=2006, timezone offset=-0700) serve as placeholders in layout strings.
- **Format**: Converts a `time.Time` into a human-readable string using a layout.
- **Parse**: Converts a string back into a `time.Time` using the same layout convention.
- **Layout constants**: Predefined strings like `time.RFC3339`, `time.RFC1123`, and `time.Kitchen` for common standards.

## Real World Context
You are building a REST API that returns timestamps in ISO 8601 format and also ingests user-submitted date strings like `"Jan 15, 2024"`. Formatting uses `t.Format(time.RFC3339)` for the outgoing JSON, and parsing uses `time.Parse("Jan 2, 2006", input)` for the incoming string. Because both operations use the same reference-time convention, there is no ambiguity about what `01` means (month) vs. `02` (day).

## Deep Dive
The mnemonic for Go's reference time is simple: 1-2-3-4-5-6-7. January is month 1, the 2nd day, at 15:04:05 (3:04:05 PM), in year 2006, in timezone MST (UTC-7). Every digit position corresponds to a component.

The standard library ships several predefined layout constants so you do not have to memorize the reference time for common formats.

```go
time.RFC3339      // 2006-01-02T15:04:05Z07:00
time.RFC1123      // Mon, 02 Jan 2006 15:04:05 MST
time.Kitchen      // 3:04PM
"2006-01-02"      // YYYY-MM-DD
"15:04:05"        // HH:MM:SS
"Jan 2, 2006"     // Mon DD, YYYY
```

These constants cover the vast majority of date/time patterns you will encounter in APIs and log files.

To format a `time.Time` into a string, call `Format` with the layout that shows the desired output shape.

```go
t := time.Now()
fmt.Println(t.Format("2006-01-02"))        // 2024-01-15
fmt.Println(t.Format(time.RFC3339))        // 2024-01-15T14:30:00Z
fmt.Println(t.Format("Monday, January 2")) // Monday, January 15
```

The output mirrors the layout but with the actual values substituted in. If you write `"2006"` in the layout, Go outputs the real year.

Parsing is the reverse: you provide the layout and the string to parse.

```go
t, err := time.Parse("2006-01-02", "2024-01-15")
t, err := time.Parse(time.RFC3339, "2024-01-15T14:30:00Z")
```

Both calls return a `time.Time` and an error. The layout must match the structure of the input string exactly, including separators and whitespace.

When the input string has no timezone information, `time.Parse` assumes UTC. If you need a specific timezone, use `time.ParseInLocation`.

```go
loc, _ := time.LoadLocation("America/New_York")
t, err := time.ParseInLocation("2006-01-02 15:04", "2024-01-15 14:30", loc)
```

This attaches the Eastern Time zone to the parsed result instead of defaulting to UTC.

## Common Pitfalls
1. **Using the wrong reference numbers** — Writing `"2006-02-01"` swaps month and day because `02` means day and `01` means month in Go's reference time. Double-check against the mnemonic: month=01, day=02.
2. **Forgetting that `time.Parse` defaults to UTC** — If your input has no timezone marker and you expect local time, use `time.ParseInLocation` instead. Otherwise your times will silently be off by your UTC offset.
3. **Using 12-hour vs. 24-hour format incorrectly** — The reference hour `15` gives 24-hour format. Use `3` for 12-hour format with `PM`.

## Best Practices
1. **Use `time.RFC3339` for machine-readable serialization** — It is the standard for APIs and JSON, includes timezone info, and round-trips cleanly through Parse.
2. **Define custom layout constants at the package level** — If your application uses a non-standard format repeatedly, declare `const myLayout = "Jan 2, 2006 at 3:04 PM"` once rather than repeating the string everywhere.

## Summary
- Go formats and parses time using a reference date (`Mon Jan 2 15:04:05 MST 2006`) instead of abstract placeholders.
- The mnemonic is 1-2-3-4-5-6-7: month 01, day 02, hour 15 (3 PM), minute 04, second 05, year 2006, timezone -0700.
- Use predefined constants like `time.RFC3339` for standard formats.
- `time.Parse` assumes UTC when no timezone is present; use `time.ParseInLocation` when you need a specific zone.
- Always double-check your layout against the reference time to avoid swapping month and day.

## Code Examples

**Formatting time using Go's reference time layout (Mon Jan 2 15:04:05 MST 2006) — each component position defines the output format**

```go
// Common formats
t := time.Now()
fmt.Println(t.Format("2006-01-02 15:04:05")) // ISO-like
fmt.Println(t.Format("Jan 02, 2006"))         // Human readable
fmt.Println(t.Format(time.RFC3339))           // Standard
```


## Resources

- [time package - Time.Format](https://pkg.go.dev/time#Time.Format) — Official reference for Go time formatting with the reference time layout

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*