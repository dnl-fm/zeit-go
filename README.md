# zeit-go

Timezone-aware time handling for Go. UTC internally, user timezone for display, integer storage for databases.

## Installation

```bash
go get github.com/dnl-fm/zeit-go
```

## Core Concept

Zeit stores time as UTC internally but preserves a timezone for display. Database storage is always `int64` Unix timestamps. User-facing output is RFC3339.

```
User (RFC3339) → Zeit (UTC) → Database (int64)
                    ↕
              Display (any TZ)
```

## Quick Start

```go
import "github.com/dnl-fm/zeit-go"

// Define app timezone once
appTZ, _ := time.LoadLocation("Europe/Berlin")

// Create
z := zeit.Now(appTZ)
z := zeit.FromUser("2024-01-15T10:30:00+01:00", appTZ)
z := zeit.FromDatabase(1705312800, appTZ)

// Convert
z.ToUser()      // "2024-01-15T10:30:00+01:00"
z.ToDatabase()  // 1705312800
z.Unix()        // 1705312800

// Switch timezone
z.In(tokyo).ToUser()  // same instant, different display
```

## Database Integration

Zeit implements `sql.Scanner` and `driver.Valuer` — use `*zeit.Zeit` in struct fields for automatic scanning:

```go
type Order struct {
    ID        string     `db:"id"`
    CreatedAt *zeit.Zeit `db:"created_at"`  // scans int64, defaults UTC
}

// After scanning, switch to user timezone
order.CreatedAt.In(appTZ).ToUser()  // "2024-01-15T11:30:00+01:00"
```

Columns must be `INTEGER` (Unix timestamp).

## Calendar Helpers

```go
z := zeit.Now(appTZ)

z.DaysInMonth()    // 31 (January)
z.DayOfMonth()     // 15
z.StartOfMonth()   // 2024-01-01T00:00:00
z.EndOfMonth()     // 2024-01-31T23:59:59
```

## Duration

Measure the distance between two moments in multiple units:

```go
start := zeit.FromUser("2024-01-01T00:00:00Z", appTZ)
end := zeit.FromUser("2024-03-15T00:00:00Z", appTZ)

d := start.Until(end)

d.Days()          // 74
d.Hours()         // 1776
d.Minutes()       // 106560
d.Seconds()       // 6393600
d.Months()        // 2
d.BusinessDays()  // 53 (Mon-Fri only)
d.Raw()           // time.Duration
```

### Proration Example

```go
// Customer subscribes Jan 1, cancels Jan 15
billingStart := zeit.FromUser("2024-01-01T00:00:00Z", tz)
cancelled := zeit.FromUser("2024-01-15T00:00:00Z", tz)

usedDays := billingStart.Until(cancelled).Days()  // 14
totalDays := cancelled.DaysInMonth()               // 31

proratedPrice := monthlyPrice * float64(usedDays) / float64(totalDays)
// 100.00 * 14/31 = 45.16
```

## Date Arithmetic

```go
z.Add(2 * time.Hour)     // add duration
z.AddDays(5)             // add calendar days
z.AddDays(-3)            // subtract days
z.AddBusinessDays(10)    // skip weekends
```

## Billing Cycles

```go
start := zeit.Now(appTZ)

// Generate 12 monthly billing periods
cycles := start.Cycles(12, zeit.Monthly)

for _, period := range cycles {
    fmt.Printf("%s → %s\n", period.StartsAt.ToUser(), period.EndsAt.ToUser())
    period.Contains(someDate)  // check if date falls within
}
```

Intervals: `zeit.Daily`, `zeit.Weekly`, `zeit.Monthly`, `zeit.Quarterly`, `zeit.Yearly`

## Comparison

```go
z1.Before(z2)  // true if z1 is earlier
z1.After(z2)   // true if z1 is later
z1.Equal(z2)   // true if same instant (ignores timezone)
```

## JSON

```go
// Marshals to RFC3339 in Zeit's timezone
data, _ := json.Marshal(z)  // "2024-01-15T10:30:00+01:00"

// Unmarshals from RFC3339, defaults to UTC
var z zeit.Zeit
json.Unmarshal(data, &z)
```

## Requirements

- Go 1.22+

## License

MIT
