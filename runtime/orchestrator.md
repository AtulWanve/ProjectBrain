# Orchestrator (Phase 8)

Every user prompt follows the same deterministic pipeline. No stage may be skipped.

## Pipeline

```
User Prompt
↓
Task Classification
↓
Graph Retrieval
↓
Context Loading
↓
Planning
↓
Execution
↓
Static Validation
└── Failure → return to Planning (no stage skip; re-execute from Planning forward)
↓ (pass)
AI Review
↓
Confidence Scoring
↓
accepted === true ?
( !blocking && overallScore >= 90 )
├── No
│   ↓
│   Retry count < max (default 3) ?
│   ├── Yes → increment retry count, return to Execution with improvement instructions
│   └── No  → escalate to human review, return escalation response (no further retries allowed)
│
└── Yes
    ↓
    Knowledge Synchronization
    ↓
    Response
```

Note: Returning to an earlier stage on validation failure or low confidence does **not** violate the no-stage-skipping rule — every stage is visited in order on each full pass through the pipeline. The re-execution path retraces all stages from the re-entry point forward.

## Responsibilities

- Enforce pipeline ordering
- Route results between stages
- Handle errors at each stage with appropriate rollback
- Ensure no stage is skipped or reordered
- Provide a single entry and exit point for all prompts
