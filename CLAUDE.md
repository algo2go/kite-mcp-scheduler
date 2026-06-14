# CLAUDE.md — kite-mcp-scheduler

`github.com/algo2go/kite-mcp-scheduler` — in-process **wall-clock task scheduler** (a single 60s ticker) for the kite-mcp engine. Drives time-of-day jobs; the consumer wires e.g. `pnl_snapshot`@15:40 IST, `audit_cleanup`@03:00, and Telegram briefings (only when a Telegram token is configured). See `../CLAUDE.md`.

## ⚠ Restart / idle-host critical finding (audit 2026-06-14)

**Tasks fire at the EXACT wall-clock minute with NO catch-up.**

- `tick()` matches `now.Hour()==t.Hour && now.Minute()==t.Minute` (`scheduler.go:205`).
- `lastRun` is an **in-memory map, reset on every boot** (`scheduler.go:39,53`) — it prevents same-day double-runs but does **not** replay a missed window. There is no "run since last execution" logic.
- The first tick fires on boot (`scheduler.go:141`), then every 60s.

**Impact:** a task only fires if the process is up during its exact target minute. On an **idle / scale-to-zero or frequently-restarted host**, a task whose minute falls inside a stopped window is **silently skipped that day**, with no observability. (Concrete case: under the hosted kite-mcp-server scale-to-zero flip, `pnl_snapshot`@15:40 is missed ~daily — analytics-only there, so accepted.)

**Recommended fix (DEFERRED, if catch-up matters):** change firing to **"now ≥ target AND lastRun[name] != today"** and **persist `lastRun`** (e.g., to the consumer's SQLite) so a boot after a missed window still runs the task once that day. This removes the dependence on wake-timing — the correct model for an idle/ephemeral host.

Full context: `../../kite-mcp-server/.research/2026-06-14-scale-to-zero-safety-audit.md` (Lens 3).
