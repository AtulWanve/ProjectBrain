# Graph Retriever

Determines the minimum context required by querying the graph.

## Output Representation

Each affected node is returned as an `AffectedNode` object:

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique node identifier matching a node in `graph/nodes.json` |
| `type` | `"module" | "api" | "route" | "component" | "config" | "test"` | Node category |
| `path` | `string` | File system path relative to project root |
| `label` | `string` | Human-readable node name |
| `connectedResources` | `{ apis: string[]; routes: string[]; components: string[] }` | Resource references discovered via graph edges |

Return type: `AffectedNode[]`.

### Empty-Graph Behavior

When `graph/nodes.json` is `[]`:

1. Return an empty `AffectedNode[]`.
2. Context Loader falls back to loading all memory files.
3. Graph initialization is **not** triggered here — it occurs via Graph Updater after the first accepted change.
