# Graph Retriever

Determines the minimum context required by querying the graph.

## Output Representation

Each affected node is returned as an `AffectedNode` object:

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique node identifier; must match a node in the current snapshot's `nodes.json`, resolved through `graph/manifest.json` via the snapshot-indirection contract |
| `type` | `"module" \| "api" \| "route" \| "component" \| "config" \| "test"` | Node category |
| `path` | `string` | File system path relative to project root |
| `label` | `string` | Human-readable node name |
| `connectedResources` | `{ apis: string[]; routes: string[]; components: string[] }` | Resource references discovered via graph edges |

Return type:

```typescript
{
  affectedNodes: AffectedNode[];
  graphEmpty: boolean;
}
```

- `graphEmpty` is `true` **only** when snapshot resolution succeeds and the parsed `nodes.json` is exactly `[]` (empty array). Resolution follows the snapshot-indirection contract: read `graph/manifest.json`, locate the current version directory (e.g., `graph/v-<timestamp>/`), and read that snapshot's `nodes.json`. It signals Context Loader to enter **Empty-Graph Fallback Mode**.
- `graphEmpty` is `false` when retrieval succeeds with any nodes. A result of zero affected nodes from a non-empty graph still returns `graphEmpty: false`.
- If manifest resolution fails (missing/unreadable `graph/manifest.json`), the referenced version directory does not exist, or `nodes.json` is missing, malformed, or fails schema validation, the retriever MUST throw a distinct error — it MUST NOT map these failures to `graphEmpty: true`. Only a successfully resolved and parsed `nodes.json` containing exactly `[]` produces `graphEmpty: true`.
- Graph initialization is **not** triggered here — it occurs via Graph Updater after the first accepted change.

### Empty-Graph Behavior

When the manifest-resolved snapshot's `nodes.json` is `[]`:

1. Return `{ affectedNodes: [], graphEmpty: true }`.
2. Context Loader enters **Empty-Graph Fallback Mode** when `graphEmpty === true`.
3. Graph initialization is **not** triggered here — it occurs via Graph Updater after the first accepted change.
