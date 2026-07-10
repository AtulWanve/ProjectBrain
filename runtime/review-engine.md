# Review Engine

Evaluates implementation quality after static validation passes.

## Structured Output Contract

### Schema

Returns a `ReviewResult` object:

| Field | Type | Description |
|-------|------|-------------|
| `overallScore` | `number` (0–100) | Aggregate quality score |
| `scores` | `ReviewDimension[]` | Per-category scores |
| `blocking` | `boolean` | If true, the change is rejected |
| `accepted` | `boolean` | Derived: `!blocking && overallScore >= 50` |
| `acceptedChange` | `AcceptedChange \| null` | Approved change payload (null when rejected) |
| `feedback` | `string` | Human-readable review summary |

### Score Meanings (`ReviewDimension`)

| Field | Type | Range |
|-------|------|-------|
| `category` | `string` | `"architecture"`, `"maintainability"`, `"readability"`, `"intent"`, `"side-effects"`, `"scalability"`, or `"regression"` |
| `score` | `number` | 0–100 |
| `weight` | `number` | 0.0–1.0 (default 0.14 per category) |
| `notes` | `string` | Justification |

### Blocking Thresholds

- Any single `score` < 40 sets `blocking = true`.
- `overallScore` is the weighted average of per-category scores.
- `overallScore` < 50 sets `blocking = true`.

### Accepted Change (`AcceptedChange`)

| Field | Type | Description |
|-------|------|-------------|
| `idempotencyKey` | `string` | UUID for safe retry (consumed by Graph Updater) |
| `taskId` | `string` | Stable task identifier (consumed by Memory Updater) |
| `graphDiff` | `{ nodes: NodeDiff[]; edges: EdgeDiff[] }` | Incremental graph changes |
| `memoryAppend` | `{ targetFile: string; content: string }[]` | Memory file append operations |

### Consumer Integration

- **Graph Updater** reads `acceptedChange.idempotencyKey` and `acceptedChange.graphDiff`.
- **Memory Updater** reads `acceptedChange.taskId` and `acceptedChange.memoryAppend`.
- Both receive the full `ReviewResult` and extract their respective fields.
