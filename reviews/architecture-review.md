# Architecture Review

## Scope

Evaluates whether the implementation aligns with the existing project architecture.

## Checklist

- [ ] Follows established project structure
- [ ] Respects module boundaries and separation of concerns
- [ ] No circular dependencies introduced
- [ ] Component decomposition is appropriate (not too large, not too granular)
- [ ] Data flow is unidirectional and predictable
- [ ] New abstractions are justified
- [ ] Backward compatible where required

## Scoring

### Checklist Item States

Each checklist item is in one of the following states:

| State | Representation | Contribution |
|-------|---------------|--------------|
| Passed | `1` | Counts toward `checks-passed` and `total-applicable` |
| Failed | `0` | Counts toward `total-applicable` only |
| Partial credit | A decimal between 0 and 1 (e.g., `0.5`) | That value counts toward `checks-passed` and toward `total-applicable` |
| N/A (not applicable) | `"N/A"` | Excluded from both `checks-passed` and `total-applicable` |

### Score Formula

Score = `(checks-passed / total-applicable) × 100`, where `total-applicable` excludes N/A items.

If all items are N/A, the score is defined as `null` (no score), `Blocking` is `false`, and the review is treated as passed for that dimension.

Threshold: score < 60 sets `Blocking: true`. Additionally, any single issue classified as "Critical" in the Issues found section sets `Blocking: true` regardless of the aggregate score. Issues are classified using the canonical severity set: `"Critical"`, `"High"`, `"Medium"`, or `"Low"`. Only `"Critical"` severity triggers blocking irrespective of score.

## Output

- Score (0–100)
- Issues found (each classified as Critical, High, Medium, or Low)
- Recommendations
- Blocking: boolean (true when score < 60 or any Critical issue is present)
