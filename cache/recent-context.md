---
title: Recent Context
tags: [cache, context, performance]
aliases: [Recent Context Cache]
---

# Recent Context

Caches the most recently loaded context to avoid redundant graph retrieval. Populated by [[runtime/context-loader|Context Loader]].

## Format

```
Task ID: <uuid>
Timestamp: <ISO-8601>
Graph Snapshot: <graph/manifest-resolved version directory, e.g., v-1712345678>
Affected Nodes: [...]
Loaded Memory Files: [...]
Context Hash: <sha256>
```

### Cache-Hit Validation

On cache lookup, `Graph Snapshot` is compared against the currently requested graph snapshot (resolved via `graph/manifest.json`). The cached entry must also cover every node and memory file requested by the current prompt, matched against `Affected Nodes` and `Loaded Memory Files`. If the snapshot does not match, or coverage is incomplete, the cache entry is treated as a miss and context is reloaded from scratch.

> [!note] Cache invalidation is driven by graph snapshot changes and requested coverage. If the [[runtime/graph-updater|Graph Updater]] publishes a new version, the snapshot will differ and the cache will miss. A matching snapshot alone is insufficient; partial coverage of the requested nodes or memory files also forces a reload.
