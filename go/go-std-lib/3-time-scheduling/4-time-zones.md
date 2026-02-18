---
source_course: "go-std-lib"
source_lesson: "go-std-lib-time-zones"
---

# Working with Time Zones

## Locations

```go
// Load a location
loc, err := time.LoadLocation("America/New_York")
loc, err := time.LoadLocation("Europe/Paris")

// Predefined locations
time.UTC
time.Local
```

## Converting Between Zones

```go
t := time.Now()  // Local time
utc := t.UTC()   // Convert to UTC
ny := t.In(loc)  // Convert to specific location
```

## Creating Time in Specific Zone

```go
loc, _ := time.LoadLocation("Asia/Tokyo")
t := time.Date(2024, 1, 15, 9, 0, 0, 0, loc)
```

## Getting Zone Info

```go
name, offset := t.Zone()
fmt.Printf("Zone: %s, Offset: %d seconds\n", name, offset)
```

## Best Practices

*   Store times in UTC in databases.
*   Convert to local time only for display.
*   Use `time.RFC3339` for serialization (includes timezone).

## Code Examples

**User-friendly time display**

```go
func formatForUser(t time.Time, userTZ string) string {
    loc, err := time.LoadLocation(userTZ)
    if err != nil {
        loc = time.UTC
    }
    return t.In(loc).Format("Jan 2, 2006 at 3:04 PM")
}
```


---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*