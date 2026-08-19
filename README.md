# Project Brain v2

## AI Runtime Architecture for Long-Term Vibecoding

---

# Vision

Project Brain is an AI-native engineering runtime that eliminates repeated codebase analysis by maintaining a continuously synchronized knowledge system.

Instead of forcing the AI to rediscover the project on every session, Project Brain provides persistent architectural memory, graph-based retrieval, deterministic execution workflows, and incremental knowledge synchronization.

The primary objective is to minimize context tokens while improving code quality through structured engineering workflows.

---

# Core Principles

Project Brain follows several non-negotiable principles:

* Never analyze the entire codebase unless explicitly requested.

* Always retrieve only the required context.

* Every prompt follows the same deterministic execution pipeline.

* Memory is modular instead of monolithic.

* Graph retrieval is the primary source of project context.

* Documentation updates automatically after accepted changes.

* Knowledge grows incrementally with every completed task.

---

# Project Structure

project-brain/

├── system/  
├── graph/  
├── memory/  
├── tasks/  
├── standards/  
├── reviews/  
├── templates/  
├── runtime/  
└── cache/

---

# Phase 1 — System Layer

The System layer defines **how the AI thinks**, not what the project contains.

## Directory

system/

[[system]]

[[planner]]

[[workflow]]

### system.md

Purpose:

Defines the global engineering workflow.

Responsibilities:

* Receive user prompts

* Enforce execution order

* Never store architecture

* Never store project memory

* Never store implementation details

This file should remain stable throughout the lifetime of the project.

---

### planner.md

Responsible for:

* Prompt decomposition

* Complexity estimation

* Risk assessment

* Task classification

* Review requirements

* Rollback strategy

---

### workflow.md

Defines the execution lifecycle. This is an abbreviated overview of the canonical pipeline in [[system/workflow]].

Receive Prompt

↓

Retrieve Context

↓

Planning

↓

Execution

↓

Static Validation

↓

Review

↓ (accepted)

Knowledge Synchronization

↓

Response

Knowledge Synchronization runs only when the review is accepted; rejected results return to Execution or escalate.

No implementation logic should exist inside this file.

---

# Phase 2 — Memory Layer

Instead of maintaining one massive memory file, Project Brain divides project knowledge into independent domains.

memory/

[[overview]]

[[architecture]]

[[backend]]

[[frontend]]

[[database]]

[[routing]]

[[api]]

[[dependencies]]

[[patterns]]

[[checkpoints]]

Each file should ideally remain under 300–500 lines.

This allows selective loading instead of injecting thousands of unnecessary tokens into context.

---

# Phase 3 — Graph Layer

The graph becomes the structural source of truth.

graph/

graph.json

graph-index.json

embeddings.json

nodes.json

edges.json

Graphify is responsible for updating these files.

The AI must never manually edit graph structures.

Graph responsibilities include:

* Module relationships

* Component hierarchy

* Route mapping

* API connections

* Database relationships

* Dependency graph

* Import/export relationships

---

# Phase 4 — Runtime Layer

The Runtime layer acts as the execution engine.

runtime/

[[orchestrator]]

[[context-loader]]

[[execution-engine]]

[[static-validator]]

[[graph-retriever]]

[[memory-updater]]

[[graph-updater]]

[[review-engine]]

[[confidence-engine]]

[[task-classifier]]

Each runtime component performs exactly one responsibility.

---

## Example Runtime Execution

User Prompt:

Add Dark Mode

Execution sequence:

Task Classifier

↓

Medium Complexity

↓

Graph Retriever

↓

Affected Nodes

* Navbar

* Theme Context

* Settings

* Theme Provider

↓

Context Loader

↓

Loads only:

* [[frontend]]

* [[architecture]]

* Theme Context

↓

Planning

↓

Execution

---

# Phase 5 — Standards Layer

Standards define engineering quality.

standards/

[[typescript]]

[[react]]

[[nextjs]]

[[security]]

[[performance]]

[[naming]]

[[documentation]]

Review agents reference standards rather than project memory.

This separates implementation knowledge from engineering rules.

---

# Phase 6 — Review Layer

reviews/

[[architecture-review]]

[[performance-review]]

[[security-review]]

[[code-quality]]

[[documentation-review]]

Each reviewer focuses on one engineering discipline.

Example:

Security Review

Checklist:

* SQL Injection

* XSS

* Authentication

* Authorization

* Secret Exposure

* Validation

* Dependency Risks

Output:

Score: 94/100

Issues Found:  
...

Recommendations:  
...

---

# Phase 7 — Task History

tasks/

[[active]]

[[completed]]

[[failed]]

[[changelog]]

Each completed task appends:

* Task Description

* Goal

* Files Changed

* Modules Changed

* Review Score

* Date

* Memory Updated

* Graph Updated

This provides long-term engineering history.

---

# Phase 8 — Orchestrator

Every user prompt follows the same deterministic pipeline.

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

↓

AI Review

↓

Knowledge Synchronization

↓

Response

No stage may be skipped; the only exception is the intentional static-validation short-circuit defined in Phase 12.

---

# Phase 9 — Graph Retrieval

The graph determines the minimum context required.

Instead of:

Analyze the project.

The runtime performs:

Find affected nodes

↓

Resolve dependencies

↓

Return connected files

↓

Return APIs

↓

Return Routes

↓

Return Components

Only relevant context enters the model.

---

# Phase 10 — Planning

Before writing code, the planner produces:

* Goal

* Affected Modules

* Execution Order

* Estimated Complexity

* Risk Analysis

* Testing Strategy

* Rollback Strategy

* Estimated Files

Only after planning is approved does execution begin.

---

# Phase 11 — Execution

The Execution Engine generates implementation changes.

Responsibilities:

* Modify code

* Respect project standards

* Avoid unrelated refactoring

* Preserve architecture

No review occurs during execution.

---

# Phase 12 — Static Validation

Before consuming AI review tokens, deterministic tooling validates the implementation.

Examples:

npm run lint

npm run typecheck

npm run build

tests

If any validation fails:

Execution returns directly to the coding phase.

AI review is skipped.

This minimizes token consumption.

A task is given a finite number of validation attempts. Until that limit is reached, retries follow the same path above. Once the limit is exhausted, retrying stops and the task is recorded in [[tasks/failed]] together with the diagnostic details.

---

# Phase 13 — AI Review

AI review only begins after static validation succeeds.

Evaluation areas:

* Architecture

* Maintainability

* Readability

* Intent Matching

* Side Effects

* Scalability

* Regression Risk

Reviewer outputs structured feedback.

---

# Phase 14 — Confidence Engine

Every accepted implementation receives quantitative scoring.

Example:

Architecture     96

Performance      91

Security         98

Naming           95

Documentation    90

Testing          92

Overall          93

Acceptance Threshold:

Overall ≥ 90

Otherwise:

Improve

↓

Review Again

This prevents unnecessary review loops.

A task may cycle through the review loop only a finite number of times. Until that limit is reached, the Improve → Review Again flow above is preserved. Once the limit is exhausted, improvement stops and the task is recorded in [[tasks/failed]] together with the review diagnostics.

---

# Phase 15 — Knowledge Synchronization

Once a task is accepted:

Only affected knowledge is updated.

Possible updates include:

* [[overview]]

* [[architecture]]

* [[routing]]

* [[api]]

* [[dependencies]]

* task history

* graph metadata

Entire documentation is never regenerated.

Only incremental changes are applied.

## Recoverable Commit Workflow

To ensure system integrity, Knowledge Synchronization follows a recoverable commit workflow:

1. **Checkpointing**: The system creates a synchronization checkpoint in [[memory/checkpoints]] before starting updates, recording the pending updates.
2. **Execution Ordering**: Updates occur in a strict order:
   - Memory updates (via [[memory-updater]])
   - Graph updates (via [[graph-updater]])
   - Task history updates
3. **Idempotency**: Both the memory and graph updaters implement idempotent retry behaviors, allowing partial updates to be safely replayed.
4. **Reconciliation**: If a failure occurs during synchronization, the system must read [[memory/checkpoints]] to resume or reconcile the incomplete updates.
5. **Completion**: A task cannot be marked complete in task history until the synchronization checkpoint is fully resolved and the updates are successful.

---

# Phase 16 — Incremental Graph Updates

Suppose a new AuthService is introduced.

Only the following graph relationships are updated:

AuthService

↓

Middleware

↓

User

↓

API

↓

Database

The remainder of the graph remains unchanged.

---

# Phase 17 — Cache Layer

cache/

[[recent-context]]

[[last-plan]]

[[recent-files]]

[[recent-review]]

Purpose:

Reduce repeated graph retrieval for sequential prompts.

Example:

Prompt 1:

Implement Authentication

Prompt 2:

Now add Forgot Password

The Runtime loads the recent authentication context directly from cache instead of rebuilding project context.

---

# Phase 18 — Complete Prompt Lifecycle

Abbreviated overview of the complete prompt lifecycle. The canonical deterministic pipeline with retry, escalation, and error handling lives in [[system/workflow]].

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

Score ≥ 90 ?

├── No  
│  
└── Improve  
      ↓  
    Review Again

↓

Yes

↓

Incremental Memory Update

↓

Incremental Graph Update

↓

Append Task History

↓

Return Final Response

Knowledge Synchronization runs only when the review is accepted; rejected reviews return to Execution or escalate.

---

# Design Philosophy

Project Brain is not a prompt.

It is an AI engineering runtime.

Its purpose is to transform large language models into persistent software engineers capable of maintaining long-term project knowledge while minimizing context usage.

Every component has a single responsibility.

Every workflow is deterministic.

Every accepted change improves the project’s collective knowledge.

Rather than repeatedly rediscovering the codebase, Project Brain continuously evolves alongside it, becoming progressively faster, more context-aware, and more reliable with every engineering session.