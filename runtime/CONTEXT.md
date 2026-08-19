# Runtime Context

This directory acts as the execution engine and orchestrator of the Project Brain pipeline, including the incremental update mechanisms (`memory-updater.md`, `graph-updater.md`).

**Rule:** Every prompt follows the deterministic pipeline defined in `orchestrator.md`. No stage may be skipped. Each file here has exactly one responsibility (e.g., retrieving context, executing code, running reviews, updating state idempotently).
