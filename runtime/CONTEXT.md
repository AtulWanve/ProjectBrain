# Runtime Context

This directory acts as the execution engine and orchestrator of the Project Brain pipeline, including the incremental update mechanisms ([[runtime/memory-updater|Memory Updater]], [[runtime/graph-updater|Graph Updater]]).

**Rule:** Every prompt follows the deterministic pipeline defined in [[runtime/orchestrator|Orchestrator]]. No stage may be skipped. Each file here has exactly one responsibility (e.g., retrieving context, executing code, running reviews, updating state idempotently).
