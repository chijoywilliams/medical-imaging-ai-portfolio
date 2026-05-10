# event-replay

Deterministic event log replay utility.

Parses newline-delimited events of the form:

```
<timestamp_ms> <event_type> <key> [value]
```

Supported event types: `set`, `inc`, `del`.

## Build & test

```
cd rust/event-replay
cargo build
cargo test
```

## Usage

```
event-replay replay   <events.log> [--lenient]
event-replay timeline <events.log> [--bucket-ms 1000] [--lenient]
event-replay snapshot <events.log> --out <snapshot.json> [--lenient]
event-replay diff     <a.log> <b.log>
```

- `replay`   — replays the log and prints the final state (sorted by key).
- `timeline` — groups events into time buckets and prints a human-readable report.
- `snapshot` — replays the log and writes the final state to a deterministic JSON file.
- `diff`     — replays both logs and prints snapshot differences.

### Determinism

Replay is deterministic. Events are sorted with a *stable* sort by
`(timestamp_ms, seq)`, where `seq` is the 0-based source-line index assigned
by the parser. Consequences:

- For two events with **different** timestamps, source order doesn't matter.
- For two events with the **same** timestamp, the line that appeared earlier
  in the source log is applied first — so the same file always replays to the
  same state, byte-for-byte.
- The snapshot JSON output is also byte-stable: keys are emitted in sorted
  (`BTreeMap`) order with typed `int` / `string` values.

### Malformed events

By default parsing is **strict**: the first malformed line aborts with a
diagnostic naming the line number and offending field. Pass `--lenient` to
skip malformed lines instead (each one is reported on stderr) and replay
whatever survives.
