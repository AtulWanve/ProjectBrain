# Memory Context

This directory stores modular knowledge about the project (architecture, routing, etc.), as well as synchronization states (`checkpoints.md`) for recoverable commits.

**Rule:** Entire documentation is never regenerated. Only incremental, affected changes are applied using [[runtime/memory-updater.md]]. Files must remain under 500 lines to prevent context bloat.
