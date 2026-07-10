# Prompt Lifecycle (Phase 18)

The complete end-to-end lifecycle of a user prompt.

## Full Pipeline

```
Receive Prompt
↓
Task Classification
↓
Graph Retrieval
↓
Load Relevant Memory
↓
Planning
↓
Execution
↓
Static Validation
↓
AI Review
↓
Confidence Scoring
↓
!blocking && overallScore >= 90 ?
├── No (blocking or score < 90)
│   ↓
│   Retry count < max (default 3) ?
│   ├── Yes → increment retry count, Improve → Review Again
│   └── No  → Escalate to human review, return escalation response (no further retries)
│
└── Yes
    ↓
    Incremental Memory Update
    ↓
    Incremental Graph Update
    ↓
    Append Task History
    ↓
    Return Final Response
```

## Design Principles

- Every stage is deterministic
- No stage may be skipped
- Failures at any stage return actionable feedback
- Knowledge grows incrementally with every completed task
- Token consumption is minimized by early validation
