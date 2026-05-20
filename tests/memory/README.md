# Memory recall test set

Curated cases that prove household memory recall is reliable and that recall
failures are observable, not silent. Tracks M1 exit criterion from
[issue #111](https://github.com/GeniePod/genie-claw/issues/111).

## Layout

- `cases.toml` — the case definitions. One `[[case]]` table per scenario.
- `expected/` — written by the runner. Contains the latest pass/fail ledger
  (`ledger.json`) so a reviewer can see which case failed without re-running.

The runner lives at `crates/genie-core/tests/memory_recall.rs` and is exercised
by `cargo test -p genie-core --test memory_recall`.

## What a case proves

Each case has a `seed`, a `query`, a `context`, and an `expect`:

| `expect.outcome` | Meaning |
| --- | --- |
| `hit`      | Recall returned at least one entry. If `contains` is set, one entry must contain that substring. |
| `filtered` | The underlying search **did** match rows, but every match was dropped by `assess_memory_read` (scope / sensitivity / spoken_policy). The user gets an empty context, but the miss is logged with `cause="policy_filtered"`. |
| `miss`     | The underlying search returned nothing. The miss is logged with `cause="no_match"`. |

`restart = true` drops and reopens `Memory` against the same SQLite file
between seed and query. That proves the recall path survives a process
restart — the M1 "next session" requirement — without spinning a binary.

## Acceptance gate

The runner asserts ≥ 95 % pass across all cases and writes
`tests/memory/expected/ledger.json` for the closing PR.
