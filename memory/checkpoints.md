# Synchronization Checkpoints

This file stores the state of ongoing knowledge synchronization operations across memory, graph metadata, and task history.

## Checkpoint Format

Checkpoints track the progress of synchronization phases to ensure recoverable commits in case of failure.

- **Task ID**: The unique identifier of the task being synchronized.
- **Status**: The current state of the sync (e.g., `PENDING`, `PARTIAL`, `COMPLETED`).
- **Completed Steps**: A list of synchronization steps that have successfully finished.
- **Pending Steps**: A list of synchronization steps that still need to run.

By persisting synchronization state here, the system can resume incomplete updates and reconcile partial failures.
