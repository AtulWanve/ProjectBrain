---
title: Last Plan
tags: [cache, planning, execution]
aliases: [Last Plan Cache]
---

# Last Plan

Caches the most recent execution plan for quick iteration. Produced by [[system/planner]], consumed by [[runtime/execution-engine|Execution Engine]].

## Format

```
Task ID: <uuid>
Goal: <description>
Affected Modules: [...]
Execution Order: [...]
Estimated Complexity: <low|medium|high>
Risk Analysis: <text>
Testing Strategy: <text>
```
