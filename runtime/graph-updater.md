# Graph Updater

Applies incremental graph updates after an accepted change.

## Synchronization Contract

The updater synchronizes five persistent files atomically in this order:

`graph/graph.json` → `graph/nodes.json` → `graph/edges.json` → `graph/graph-index.json` → `graph/embeddings.json`

### Update Ordering

1. Write `graph.json` (full graph state)
2. Write `nodes.json` (node list)
3. Write `edges.json` (edge list)
4. Write `graph-index.json` (lookup index)
5. Write `embeddings.json` (vector index)

### Transactional Commit / Rollback

- Writes go to temp files (`*.tmp`) first.
- Only after all five succeed are temp files atomically renamed over originals.
- If **any** write fails, all temp files are deleted; persistent files are unchanged.
- Operation is all-or-nothing — no partial updates are visible.

### Stale-Index Recovery

- On startup, if `graph-index.json` or `embeddings.json` is `{}` but `nodes.json` has entries, rebuild both indices from `nodes.json` and `edges.json`.
- Recovery runs before any update operation.

### Idempotency Key

- Every accepted change carries an `idempotencyKey: string` (UUID).
- The updater records the last-applied key in `graph-index.json` metadata (`.lastAppliedKey`).
- If the same key is presented again, skip all writes and return success immediately.
- This enables safe retry without duplicating graph entries.
