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

On cache lookup, `Graph Snapshot` is compared against the currently requested graph snapshot (resolved via `graph/manifest.json`). If they do not match, the cache entry is treated as a miss and context is reloaded from scratch.

> [!note] Cache invalidation is driven by graph snapshot changes. If the [[runtime/graph-updater|Graph Updater]] publishes a new version, the snapshot will differ and the cache will miss.
