# Graph Updater

Applies incremental graph updates after an accepted change.

## Synchronization Contract

The updater synchronizes six persistent files under a single manifest switch in this order:

`graph/graph.json` → `graph/nodes.json` → `graph/edges.json` → `graph/graph-index.json` → `graph/embeddings.json` → `graph/applied-keys.ndjson`

### Update Ordering

1. Write `graph.json` (full graph state)
2. Write `nodes.json` (node list)
3. Write `edges.json` (edge list)
4. Write `graph-index.json` (lookup index)
5. Write `embeddings.json` (vector index)
6. Write `applied-keys.ndjson` (idempotency key set)

### Transactional Commit / Snapshot Manifest

Cross-file atomicity via sequential renames is **not** possible — a reader could observe a partially-updated graph between renames. Instead:

1. Write all six files to a new version directory (e.g., `graph/v-<timestamp>/`).
2. Write a single manifest file `graph/manifest.json` pointing to the current version directory.
3. After all files succeed, atomically write the manifest (single-file write; use a temp file + rename).
4. On read, each consumer resolves `graph/manifest.json` to locate the current graph snapshot.
5. If **any** file write fails, delete the incomplete version directory; the manifest still points to the previous valid snapshot.
6. Readers always see a single committed revision via the manifest pointer.

This guarantees all-or-nothing visibility: either the manifest points to a complete version or the previous version.

### Crash Durability

Each snapshot file must be flushed (fsync) after writing, before the manifest is published:

1. Write each file to the new version directory, then `fsync` the file handle.
2. After all snapshot files are written and flushed, `fsync` the version directory itself (`graph/v-<timestamp>/`) to ensure directory entries for the new files are durable. **If this fsync fails on a platform where directory fsync is supported (POSIX), treat it as a fatal error: delete the incomplete version directory, abort the publication, and preserve the existing manifest. The caller may retry from scratch.**
3. `fsync` the parent `graph/` directory to make the version directory entry durable before referencing it in the manifest. **On platforms where directory fsync is supported (POSIX), failure at this step is fatal: abort publication, delete the incomplete version directory, preserve the existing manifest. On Windows (where directory fsync is not supported), treat this step as a no-op.**
4. Write the manifest to a temporary file (e.g., `graph/manifest.json.tmp`) and `fsync` it.
5. `rename` the temporary manifest to `graph/manifest.json`.
6. `fsync` the containing directory (`graph/`) after the rename.

On **Windows**, `fsync` is available via `_commit` (C) or `FileStream.Flush(true)` (.NET); `fsync` on directories is not supported — directory durability after rename and after the version-directory step is not guaranteed on all platforms. **Therefore, the version-directory fsync (step 2) SHOULD be skipped or treated as a no-op on Windows; a failure or missing implementation at this step MUST NOT block manifest publication, because the platform provides no mechanism for this guarantee.** On **POSIX**, `fsync(fd)` flushes the file data and `fsync(dir_fd)` (or `O_SYNC` directory handle) flushes the directory entry, and a failure at step 2 is treated as fatal per the note above. Implementations should document platform-specific limitations and accept best-effort crash durability on platforms where directory fsync is unavailable.

### Stale-Version Cleanup

- Keep only the two most recent version directories for recovery.
- Older versions are pruned on the next successful commit.

### Stale-Index Recovery

On startup, if the manifest version's `graph-index.json` is `{}` and its `nodes.json` has entries, or if `embeddings.json` contains no non-reserved vector entries (i.e., every key in `embeddings.json` begins with `_`) and `nodes.json` has entries:

1. Acquire the graph lock (`graph/.graph-lock`). This MUST happen before entering the try block. The entire recovery sequence that follows (including embedding generation, incomplete-directory cleanup, and manifest publication) MUST guarantee lock release on every exit path — success, abort, provider failure, cleanup failure, or manifest error. Implementations MUST use a try/finally or equivalent construct to enforce this, with lock acquisition before try and exactly one unlock call in the finally block.
2. **Rebuild `graph-index.json`** deterministically from `nodes.json` and `edges.json`.
3. **Regenerate `embeddings.json`** using the authoritative embedding provider and model recorded in the previous snapshot's embedding metadata (or from a configuration-defined default model). The snapshot may store embedding metadata inside `embeddings.json` using `_`-prefixed reserved keys (e.g., `{"_embedding_model": {"provider": "openai", "model": "text-embedding-3-small", "version": "2024-01"}}`). All keys starting with `_` in `embeddings.json` are reserved metadata; vector consumers MUST ignore them. If no prior embedding metadata exists, use the configured default; if neither is available, **abort recovery** — do not release the lock directly (the enclosing finally block handles release), leaving the current manifest unchanged.
4. Write all regenerated files into a **new** version directory (e.g., `graph/v-<timestamp-recovery>/`). Include copies of all other snapshot files so the new directory is a complete atomic revision.
5. **Publish the new manifest version** using the complete durability sequence from steps 4–6 above (temp file → `fsync` → rename → `fsync` parent directory): write to a temp file, `fsync` it, rename to `graph/manifest.json`, then `fsync` the containing `graph/` directory. This MUST happen only after all files in the new revision are ready **and** vectors were successfully generated. If embedding regeneration fails (provider unavailable, model version mismatch), delete the incomplete version directory and **abort** — do not release the lock directly (the enclosing finally block handles release), and do not publish a manifest pointing to a revision with missing or invalid vector data.
6. The lock is released by the enclosing finally block after all steps complete or any abort occurs.

Recovery never mutates the currently readable manifest version in place — it produces a new complete revision and publishes it atomically. Recovery runs before any update operation.

### Idempotency Key (Versioned Snapshot Applied-Key Set)

- Every accepted change carries an `idempotencyKey: string` (UUID).
- Applied-key state is stored **exclusively within the versioned snapshot** as `graph/v-<timestamp>/applied-keys.ndjson`. No separate commit journal is maintained — the applied-key set in the snapshot is the sole authoritative representation.
- On each update:
  1. Acquire an advisory file lock (`graph/.graph-lock`).
  2. Read the current manifest to obtain the latest committed revision and its applied-key set.
  3. Check if `idempotencyKey` already exists in the applied-key set.
     - If **found**: skip all writes, return success immediately.
     - If **not found**: proceed with writes.
   4. Write the graph snapshot (including updated `applied-keys.ndjson`) to a **new** version directory.
   5. **Atomically** publish the manifest: write to a temp file, `fsync` it, rename to `graph/manifest.json`, then `fsync` the containing directory (`graph/`). The manifest is published only after `applied-keys.ndjson` and all other snapshot files are successfully flushed. No post-rename revision validation is performed — the advisory lock guarantees writer serialization, and the atomic rename guarantees all-or-nothing visibility. If a concurrent writer held the lock, step 1 would have blocked; after acquiring the lock, no other writer can interfere.
  6. Release the lock.
- A key is considered recorded **only** when the manifest durably points to the snapshot containing it. A failed or interrupted manifest write leaves no trace of the key in the applied-key set. This ensures key persistence and manifest publication are recovered atomically.
- Retries must re-read the current manifest before applying changes, ensuring the latest applied-key set is always consulted.
- Locking ensures all operations are serialized — no two concurrent commits can process the same key.
- This enables safe retry of **any** previously accepted change, not only the most recent one.

### Applied-Key Retention and Compaction

- **Maximum set size:** The applied-key set SHOULD NOT exceed 10,000 entries. When a new key would cause the set to exceed this limit, the oldest entries (by insertion order, with insertion timestamp recorded per key) are evicted before the new key is appended.
- **Eviction semantics:** Once a key is evicted from the applied-key set, a subsequent retry of the same idempotency key is treated as a **new change** — deduplication does not apply. Consumers that require indefinite deduplication MUST prune the set on a schedule and accept the window of re-execution risk after eviction.
- **Compaction trigger:** Compaction runs as part of each commit. After reading the current applied-key set, before checking the new key:
  1. If set size ≥ 10,000 entries, discard entries in insertion order until size < 8,000.
  2. If any entry's insertion timestamp exceeds 90 days (configurable), discard all entries older than the threshold.
  3. Both conditions are evaluated independently; if either fires, evicted keys are removed before the idempotency check runs.
- **Authoritative representation:** The applied-key set is stored as a **newline-delimited JSON (NDJSON)** file at `graph/v-<timestamp>/applied-keys.ndjson`. Each line is a JSON object with the following schema:
  ```json
  {"key": "<UUID v4>", "ts": "<ISO-8601 UTC>"}
  ```
  Lookups scan the file sequentially (enabled by the bounded size of ≤10,000 entries). For deployments exceeding 10,000 entries, implementations MAY substitute an embedded indexed store (e.g., SQLite, btree-backed file) as `applied-keys.db` with the same durability contract; the store file is part of the versioned snapshot and published atomically via the manifest pointer. Deployments using `"db"` format MUST ensure all readers implement the indexed store format; readers that do not support `"db"` MAY reject the snapshot with a clear error.
- **Manifest format discriminator:** The manifest (`manifest.json`) MUST include an `appliedKeysFormat` field indicating the applied-keys format for the referenced snapshot. Valid values: `"ndjson"` (authoritative), `"json"` (legacy JSON array), `"db"` (indexed store). Readers MUST resolve this field to determine how to parse the applied-keys file. If the manifest lacks an `appliedKeysFormat` field, readers MUST fall back to checking for `applied-keys.json` in the snapshot directory — if it exists, treat the format as legacy `"json"`; otherwise treat the missing field as a recoverable error and follow the migration contract below.
- **Legacy support:** Readers MUST support the legacy JSON array format (`applied-keys.json`, array of `{"key": "<uuid>", "ts": "<ISO-8601>"}` objects), the authoritative NDJSON format (`applied-keys.ndjson`), and, when implemented, the indexed store format (`applied-keys.db`). The manifest `appliedKeysFormat` field selects the format; when absent, the presence of `applied-keys.json` determines the format as described above.
- **Migration contract:** A writer that reads a snapshot using a non-NDJSON format MUST convert the applied-key set to NDJSON before writing a new revision:
  1. Load the full set from the source format into memory.
  2. Write it as NDJSON lines adhering to the schema above.
  3. Publish the manifest with `appliedKeysFormat: "ndjson"` via the normal atomic procedure (temp file + fsync + rename).
  Migration never mutates a previously published snapshot in place — it always produces a new versioned revision. A crash during migration leaves the prior snapshot (and its format) intact, preserving atomic visibility.
- **Durability guarantee:** The same fsync sequence applies: the applied-key file (or store) is flushed before the manifest is published. Eviction does not alter previously published snapshots — it only affects the next committed revision. A crash between eviction and manifest publication leaves the prior snapshot (including the full pre-eviction key set) intact.
