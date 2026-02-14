# github.com/dnl-fm/zeit-go

## Stack

- go

## Commands

```bash
# Test
make test

# Lint
make lint
```

## Structure

Single-package library at root level.

| File | Description |
|------|-------------|
| `zeit.go` | Core type, constructors, Scanner/Valuer, calendar helpers |
| `duration.go` | Duration between two Zeit instances (Days, Months, BusinessDays) |
| `billing.go` | Billing cycles and periods |
