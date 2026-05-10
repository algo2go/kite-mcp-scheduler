# kite-mcp-scheduler

[![Go Reference](https://pkg.go.dev/badge/github.com/algo2go/kite-mcp-scheduler.svg)](https://pkg.go.dev/github.com/algo2go/kite-mcp-scheduler)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Goroutine-safe scheduler for the algo2go ecosystem. Provides
cron-style + interval ticker primitives with Clock injection for
deterministic testing, lifecycle management (Start/Stop), and
goroutine-leak sentinels.

Used by [`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
for IST-aligned background dispatch (morning briefings, EOD P&L,
audit-log retention sweeps, Telegram dispatch loops, alert evaluation
ticks).

## Why a separate module?

Background scheduling is an orthogonal infrastructure primitive —
usable by any algo2go project (broker dashboards, monitoring,
broker-adapter health checks, future trading bots) independent of
`kite-mcp-server`. Centralizing as a module:

- Lets the Clock-injection pattern + leak sentinels be shared
  consistently across consumers
- Encourages deterministic-test discipline (Clock injection +
  goroutine-leak detection together)
- Keeps the dep-graph weight minimal for users who only need
  scheduling

## Stability promise

**v0.x — unstable.** Public types may break between minor versions.
Pin `v0.1.0` deliberately. v1.0 ships only after the public API is
reviewed for stability.

## Install

```bash
go get github.com/algo2go/kite-mcp-scheduler@v0.1.0
```

## Dependencies

- `github.com/algo2go/kite-mcp-isttz` — IST timezone wrapper for
  market-hours-aligned ticks
- `go.uber.org/goleak` — goroutine-leak detection in tests
- `github.com/stretchr/testify` — assertions

## Public API (scheduler.go)

- `Scheduler` — lifecycle-managed scheduler with Start/Stop semantics
- `Clock` interface — Now() + tick scheduling; for deterministic tests
  inject `MockClock`
- Tick registration helpers: every-N-minutes, daily-at-IST,
  weekday-at-IST patterns

See pkg.go.dev for full type docs.

## Reference consumer

[`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
— wires Scheduler in `app/providers/scheduler.go` and consumes via:
- Morning briefing dispatch (9:00 IST weekdays)
- EOD P&L snapshot (15:35 IST weekdays)
- Alert evaluation tick (every 30s during market hours)
- Audit-log retention sweep (3:00 IST daily)

## License

MIT — see [LICENSE](LICENSE).

## Authors

Original design: [Sundeepg98](https://github.com/Sundeepg98) (Zerodha
Tech). Multi-module promotion (2026-05-10): algo2go contributors.
