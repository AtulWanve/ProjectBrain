# Memory Updater

Applies incremental knowledge updates after an accepted change.

## Safe Append Semantics

### Stable Unique Identifier

Each task result carries a stable `taskId: string` (UUIDv4) assigned by the planner. This identifier is preserved across retries — the same task always produces the same `taskId`.

### Deduplication (Lock-Guarded)

The entire deduplication-and-replace sequence must be protected by `.memory-lock`:

1. Acquire `memory/.memory-lock` (advisory lock via `flock` / `LockFile`, 5-second timeout).
2. Scan the target `memory/*.md` file for an existing `<!-- taskId: <id> -->` marker.
3. If **found**: release lock and skip (idempotent retry).
4. If **not found**: proceed to Atomic Writes below, then release lock **only after** the rename succeeds.
5. If any step fails, release lock and propagate the error.

Holding the lock across scan → write → rename prevents concurrent workers from interleaving and creating duplicate or lost entries.

### Atomic Writes

1. Create the temp file (`*.tmp`) on the **same filesystem** as the target file — cross-filesystem moves are not atomic.
2. Write new content to the temp file.
3. Rename (replace) the temp file over the original using an atomic rename primitive (`rename()` / `MovFileEx`).
4. If write or rename fails, the original file is unchanged.
5. **Operation-time guarantee only**: This sequence prevents partial updates from concurrent or failed operations within the lifetime of the process. Durability guarantees are platform- and filesystem-specific:

   - **POSIX**: `rename()` is atomic on the same filesystem. Calling `fsync` on the temp file before rename and on the parent directory after rename provides best-effort crash durability. However, `fsync` on a directory does **not** universally guarantee directory-entry durability across all POSIX filesystems (ext4 with default options generally provides it; FUSE, NFS, and some journaling modes may not).
   - **Windows**: `MoveFileEx` with `MOVEFILE_REPLACE_EXISTING | MOVEFILE_WRITE_THROUGH` provides atomic replacement on NTFS. Without `MOVEFILE_WRITE_THROUGH`, the system may fall back to a copy+delete sequence. Calling `FlushFileBuffers` on the temp file before rename and on the parent directory handle after rename is required for power-loss recovery on NTFS, but directory `FlushFileBuffers` is not supported on all Windows configurations and may be a no-op on non-NTFS volumes (FAT32, exFAT, network shares).
   - **Failure behavior**: If the pre-rename flush or the post-rename parent-directory flush is unavailable or fails, the operation degrades to an operation-time guarantee only — the original file remains unchanged on failure, but power-loss recovery is not assured.
   - **Testing requirement**: Before guaranteeing unchanged originals or power-loss recovery in production, implement and pass crash/failure tests that simulate power loss at each step of the sequence (after temp write, after flush, after rename, after parent-dir flush). See ["Operation-time guarantee only"](#operation-time-guarantee-only) above.

### Conflict Handling for Concurrent Updates

- The `.memory-lock` serializes all write operations (see Deduplication above).
- If the lock cannot be acquired within 5 seconds, fail with `ConcurrentUpdateError`.
- Locked operations prevent duplicate entries and data loss.
