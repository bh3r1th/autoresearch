# Preflight Intent Governor (Integration)

This fork integrates a deterministic pre-execution guard that prevents repeated failed
parameter configurations from executing.

The goal is simple:
> Do not repeat known failures.

---

## What is added

- Preflight check before training
- Post-execution outcome recording
- Parameter-level failure memory
- Deterministic blocking of repeated failed values

No retries. No heuristics. No ML.

---

## How it works

### 1. Preflight (before execution)
- Extract parameters from the run
- Normalize values
- Build `block_id`
- Check history (JSONL)

Decision:
- `allow_no_conflict`
- `block_repeated_failed_value`
- `allow_new_direction`

If blocked → execution stops immediately.

---

### 2. Post-execution (after run)
- Record outcome (`success` / `failure`)
- Attach failure tags
- Append to history

---

## History format (simplified)

Each row represents a parameter attempt:

```json
{
  "run_id": "fail-1",
  "block_id": "blk_...",
  "param_name": "eps",
  "param_value_normalized": -1e-10,
  "outcome": "failure",
  "failure_tags": ["runtime_error"]
}
```

## Validated behavior
Run 1: eps = -1e-10 → FAIL → recorded
Run 2: eps = -1e-10 → BLOCKED (preflight)
Run 3: eps = 1e-10  → ALLOWED → SUCCESS

This proves:
* failure memory
* deterministic blocking
* safe exploration of new values

## Enable governor
```
AUTORESEARCH_GOVERNOR_ENABLED=1 \
AUTORESEARCH_GOVERNOR_HISTORY_PATH=./governor.jsonl \
AUTORESEARCH_GOVERNOR_RUN_ID=<run_id> \
uv run train.py
```

## Key properties
* Deterministic (no randomness)
* Parameter-level (not run-level)
* Append-only history
* Fail-fast (no wasted compute)

## Scope
This is a guard layer, not:
* an evaluator
* a retry system
* an optimizer

It enforces: “Do not execute configurations that are already known to fail.”
