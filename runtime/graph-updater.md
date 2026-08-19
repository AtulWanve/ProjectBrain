# Graph Updater

The Graph Updater is responsible for applying incremental structural changes to the graph files in `graph/`.

## Idempotency Contract

- The graph updater is strictly idempotent.
- Graph updates specify the nodes and edges to be added, removed, or modified.
- Operations like `addNode` or `addEdge` will not duplicate elements if they already exist with the same properties.
- Operations like `removeNode` or `removeEdge` will silently succeed if the element does not exist.
- If a synchronization job fails midway, restarting it will safely replay the operations without corrupting the graph state.
