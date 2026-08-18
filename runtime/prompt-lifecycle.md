---
title: Prompt Lifecycle
tags: [runtime, lifecycle, pipeline, end-to-end]
aliases: [PromptLifecycle, Phase 18]
---

# Prompt Lifecycle (Phase 18)

The complete end-to-end lifecycle of a user prompt. See also [[runtime/orchestrator]] for the deterministic pipeline and [[system/workflow]] for the high-level view.

## Full Pipeline

```
Receive Prompt
↓
Task Classification      ← [[runtime/task-classifier|Task Classifier]]
↓
Graph Retrieval          ← [[runtime/graph-retriever|Graph Retriever]]
↓
Load Relevant Memory     ← [[runtime/context-loader|Context Loader]]
↓
Planning                 ← [[system/planner]]
↓
Execution                ← [[runtime/execution-engine|Execution Engine]]
↓
Static Validation        ← [[runtime/static-validator|Static Validator]]
↓
AI Review                ← [[runtime/review-engine|Review Engine]]
↓
Confidence Scoring       ← [[runtime/confidence-engine|Confidence Engine]]
↓
!blocking && (overallScore >= 90 || reviewRequired === false) ?
├── No
│   ↓
│   Retry count < max (default 3) ?
│   ├── Yes → increment retry count, return to Execution with improvement instructions
│   └── No  → Escalate to human review, return escalation response (no further retries)
│
└── Yes
    ↓
    Incremental Memory Update   ← [[runtime/memory-updater|Memory Updater]]
    ↓
    Incremental Graph Update    ← [[runtime/graph-updater|Graph Updater]]
    ↓
    Append Task History         → updated in [[tasks/completed]] / [[tasks/failed]]
    ↓
    Return Final Response
```

## Design Principles

- Every stage is deterministic
- No stage may be skipped
- Failures at any stage return actionable feedback
- Knowledge grows incrementally with every completed task
- Token consumption is minimized by early validation

> [!info] This lifecycle is enforced by the [[runtime/orchestrator|Orchestrator]]. Each runtime component runs in strict order — no stage-skipping is permitted even on retries.
