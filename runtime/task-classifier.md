# Task Classifier

Classifies incoming user prompts before graph retrieval.

## Responsibilities

- Prompt decomposition
- Complexity estimation
- Risk assessment
- Task type classification
- Determine review requirements
- Define rollback strategy

## Output Contract

Returns a `ClassifierResult` object:

| Field | Type | Description |
|-------|------|-------------|
| `taskType` | `"feature" \| "fix" \| "refactor" \| "docs" \| "test" \| "config" \| "other"` | Category of the task |
| `complexity` | `"low" \| "medium" \| "high"` | Estimated implementation complexity |
| `risk` | `"low" \| "medium" \| "high"` | Risk of unintended side effects |
| `reviewRequired` | `boolean` | Whether AI review is mandatory. When `true`, the orchestrator must run AI Review and the result must pass the acceptance gate (`!blocking && overallScore >= 90`) before proceeding. When `false`, AI Review still runs (it is never skipped for classified tasks) but a non-passing review does **not** block acceptance — the orchestrator treats `reviewRequired: false` as permission to proceed even if the review score is below threshold, unless other pipeline stages produce a blocking failure. |
| `rollbackStrategy` | `"none" \| "revert" \| "migration"` | Strategy for undoing the change if needed |
| `confidence` | `number` (0–100) | Classifier's confidence in its own assessment |
