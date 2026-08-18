---
title: Planner
tags: [system, planner, planning, classification]
aliases: [Planning Phase]
---

# Planner

Part of the pipeline orchestrated by [[runtime/orchestrator|Orchestrator]]. Responsible for:

- Prompt decomposition
- Complexity estimation
- Risk assessment
- Task classification — uses [[runtime/task-classifier|Task Classifier]]
- Review requirements
- Rollback strategy

> [!note] The Planner runs after [[runtime/context-loader|Context Loader]] has assembled the relevant memory files. Its output feeds into the [[runtime/execution-engine|Execution Engine]].
