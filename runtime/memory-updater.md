# Memory Updater

Applies incremental knowledge updates after an accepted change.

## Safe Append Semantics

### Stable Unique Identifier

Each task result carries a stable `taskId: string` (UUIDv4) assigned by the planner. This identifier is preserved across retries — the same task always produces the same `taskId`.

### Deduplication

Before appending, scan the target `memory/*.md` file for an existing `<!-- taskId: <id> -->` marker. If found, skip the append (idempotent retry). If not found, append the task result with a new marker.

### Atomic Writes

1. Write new content to a temp file (`*.tmp`).
2. Rename temp file over the original.
3. If write or rename fails, the original file is unchanged.

### Conflict Handling for Concurrent Updates

- An advisory file lock (`memory/.memory-lock`) serializes writes via `flock` / `LockFile`.
- If the lock cannot be acquired within 5 seconds, fail with `ConcurrentUpdateError`.
- Locked operations prevent duplicate entries and data loss.
