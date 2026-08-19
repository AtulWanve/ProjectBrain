# Project Brain Context

This project implements **Project Brain v2**, an AI-native engineering runtime.

## Core Philosophy
The objective is to minimize context tokens while improving code quality through structured engineering workflows. Rather than repeatedly rediscovering the codebase, the runtime continuously evolves alongside it.

## Execution Flow
The root pipeline logic is governed by the `system/` and `runtime/` directories.
1. Task classification
2. Graph retrieval (minimizing loaded context)
3. Context loading
4. Planning
5. Execution
6. Static Validation
7. AI Review & Confidence Scoring
8. Incremental Knowledge Update

For detailed architectural intent, see [[Project Brain v2.md]].
