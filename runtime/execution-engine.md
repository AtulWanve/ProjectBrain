---
title: Execution Engine
tags: [runtime, execution, implementation]
aliases: [ExecutionEngine]
---

# Execution Engine

Generates implementation changes based on the approved plan. Runs after [[system/planner|Planning]], feeds into [[runtime/static-validator|Static Validator]].

## Responsibilities

- Modify code according to the plan
- Respect [[standards/index|project standards]]
- Avoid unrelated refactoring
- Preserve existing architecture
- No review occurs during execution

## Session Memory (Blueprint)

For long-running execution sessions, consider using a [ChatIndex](https://github.com/VectifyAI/ChatIndex)-style **topic tree** to track conversation/execution history:

- Build a hierarchical topic tree as the session progresses
- LLM-guided retrieval loads only relevant branches into context
- Enables long-running sessions without context rot

See [[memory/vectifyai-blueprints]] for the full pattern specification.

> [!important] The Execution Engine must follow [[standards/documentation|Documentation Standards]], [[standards/naming|Naming Standards]], [[standards/typescript|TypeScript Standards]], [[standards/react|React Standards]], [[standards/security|Security Standards]], and [[standards/performance|Performance Standards]].
