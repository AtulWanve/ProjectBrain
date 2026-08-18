---
title: Orchestrator
tags: [runtime, orchestrator, pipeline, core]
aliases: [Pipeline Orchestrator, Phase 8]
---

# Orchestrator (Phase 8)

Every user prompt follows the same deterministic pipeline. No stage may be skipped. See also [[system/workflow]] for the high-level lifecycle and [[runtime/prompt-lifecycle]] for the end-to-end prompt journey.

## Pipeline

```
User Prompt
↓
Task Classification    ← [[runtime/task-classifier|Task Classifier]]
↓
Graph Retrieval        ← [[runtime/graph-retriever|Graph Retriever]]
↓
Context Loading        ← [[runtime/context-loader|Context Loader]]
↓
Planning               ← [[system/planner]]
↓
Execution              ← [[runtime/execution-engine|Execution Engine]]
↓
Static Validation      ← [[runtime/static-validator|Static Validator]]
└── Failure → return to Execution (no stage skip; re-execute from Execution forward)
↓ (pass)
AI Review              ← [[runtime/review-engine|Review Engine]]
↓
Confidence Scoring     ← [[runtime/confidence-engine|Confidence Engine]]
↓
accepted === true ?
( !blocking && (overallScore >= 90 || reviewRequired === false) )
├── No
│   ↓
│   Retry count < max (default 3) ?
│   ├── Yes → increment retry count, return to Execution with improvement instructions
│   └── No  → escalate to human review, return escalation response (no further retries allowed)
│
└── Yes
    ↓
    Knowledge Sync     ← [[runtime/memory-updater|Memory Updater]] + [[runtime/graph-updater|Graph Updater]]
    ↓
    Response
```

> [!note] Returning to an earlier stage on validation failure or low confidence does **not** violate the no-stage-skipping rule — every stage is visited in order on each full pass through the pipeline. The re-execution path retraces all stages from the re-entry point forward.

## Responsibilities

- Enforce pipeline ordering
- Route results between stages
- Handle errors at each stage with appropriate rollback
- Ensure no stage is skipped or reordered
- Provide a single entry and exit point for all prompts
