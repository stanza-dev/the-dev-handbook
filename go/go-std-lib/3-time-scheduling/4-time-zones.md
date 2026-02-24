---
source_course: "go-std-lib"
source_lesson: "go-std-lib-time-zones"
---

# Time Zones & Locations

## Introduction
Time zones are one of the most bug-prone areas in software. Daylight saving transitions, political zone changes, and ambiguous local times catch even experienced developers off guard. Go's `time.Location` type and the IANA timezone database give you the tools to handle these correctly, as long as you follow a few strict conventions.

## Key Concepts
- **time.Location**: A Go type representing a geographic timezone, loaded from the IANA Time Zone database (e.g., `"America/New_York"`, `"Europe/Paris"`).
- **time.UTC**: The predefined UTC location, with no daylight saving transitions and no offset.
- **time.Local**: The location of the machine's local timezone, determined by the OS.
- **IANA Time Zone database**: The authoritative source for timezone rules, updated several times a year to reflect political changes.

## Real World Context
You are building a scheduling feature that lets users in different countries set reminders for specific times. A user in Paris sets a reminder for 9:00 AM during winter (CET, UTC+1). When summer arrives, France switches to CEST (UTC+2). If you stored the reminder as a fixed UTC offset, it would fire at the wrong local time after the DST switch. By storing the IANA location `"Europe/Paris"` and applying it at trigger time, Go's timezone database handles the transition automatically.

## Deep Dive
You load a location from the IANA database using `time.LoadLocation`. Go also provides two built-in locations.

```go
loc, err := time.LoadLocation("America/New_York")
loc, err := time.LoadLocation("Europe/Paris")

time.UTC   // Predefined: no offset
time.Local // Predefined: machine's local zone
```

`LoadLocation` returns an error if the timezone name is invalid or the timezone database is not available on the system. Always check the error.

Converting an existing `time.Time` between zones preserves the instant but changes the displayed clock values.

```go
t := time.Now()  // Local time
utc := t.UTC()   // Convert to UTC
ny := t.In(loc)  // Convert to specific location
```

All three variables represent the same instant. The difference is how they display: `utc` shows no offset, `ny` shows the Eastern Time offset. Internally, the underlying seconds since epoch are identical.

You can also create a time directly in a specific zone.

```go
loc, _ := time.LoadLocation("Asia/Tokyo")
t := time.Date(2024, 1, 15, 9, 0, 0, 0, loc)
```

This creates 9:00 AM Japan Standard Time. Converting it to UTC gives midnight of the same day.

You can inspect the zone name and UTC offset of any `time.Time`.

```go
name, offset := t.Zone()
fmt.Printf("Zone: %s, Offset: %d seconds\n", name, offset)
```

The offset is in seconds east of UTC. For example, `America/New_York` during EST returns -18000 (5 hours behind UTC).

## Common Pitfalls
1. **Storing local time in databases** — If you store `"2024-03-10 02:30:00"` without a timezone, you lose information about which instant it represents. Worse, that time might not exist in zones that spring forward at 2:00 AM. Always store UTC.
2. **Using fixed offsets instead of IANA names** — Storing `"UTC-5"` for New York ignores daylight saving time. During summer, New York is UTC-4. Use `"America/New_York"` and let Go resolve the correct offset.
3. **Assuming `time.Local` is consistent across environments** — Your development machine might be in `Europe/Paris`, but your production server is in `UTC`. Code that relies on `time.Local` produces different results in different environments.

## Best Practices
1. **Store all times in UTC in your database** — Convert to the user's local timezone only at the display layer. This eliminates ambiguity and makes cross-timezone queries straightforward.
2. **Use `time.RFC3339` for serialization** — It includes the timezone offset, so the receiver always knows the exact instant. It round-trips cleanly through `time.Parse`.

## Summary
- `time.Location` represents a timezone loaded from the IANA database via `time.LoadLocation`.
- `t.In(loc)` and `t.UTC()` convert between zones without changing the underlying instant.
- Always store times in UTC in your database; convert to local time only for display.
- Use IANA timezone names (`"America/New_York"`) instead of fixed offsets (`"UTC-5"`) to handle DST automatically.
- Serialize with `time.RFC3339` to preserve timezone information across systems.

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


## Resources

- [time package - Location](https://pkg.go.dev/time#Location) — Official reference for time zones and location handling in Go

---

> 📘 *This lesson is part of the [Go Standard Library Mastery](https://stanza.dev/courses/go-std-lib) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*