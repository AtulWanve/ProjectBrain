# Recent Review

Caches the most recent review result for iteration context.

## Format

```
Task ID: <uuid>
Review Score: <integer 0-100>
Blocking Issues: [<string>, ...]  or [] when none
Feedback: <text>
Improvement Areas: [<string>, ...]  or [] when none
```

- `Review Score` must be an integer from 0 through 100 (inclusive).
- `Blocking Issues` serializes as a JSON array of strings, or the literal `[]` when none.
- `Improvement Areas` serializes as a JSON array of strings, or the literal `[]` when none.
