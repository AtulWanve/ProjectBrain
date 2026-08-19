# Memory Updater

The Memory Updater is responsible for applying incremental updates to the modular memory files in `memory/`.

## Idempotency Contract

- The memory updater operates idempotently by using task IDs and content hashes as update anchors.
- Before applying an update, it checks if the exact same change has already been applied for the given task.
- Partial updates are designed so that re-applying them over a partially updated file results in the same final state.
- Updates are strictly incremental.
