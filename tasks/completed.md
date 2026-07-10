# Completed Tasks

Records all successfully completed engineering tasks.

## Format

```
- [Task ID] | Task Description | Goal | Files Changed | Modules Changed | Review Score (0–100) | Date (YYYY-MM-DD) | Memory Updated (true/false) | Graph Updated (true/false)
```

**Example:**
```
- T-042 | Implement login flow | Add OAuth2 authentication to the app | src/auth/login.tsx, src/auth/callback.ts | Auth Module | 92 | 2026-07-10 | true | true
```

- `Review Score` must be an integer 0–100.
- `Date` must use `YYYY-MM-DD` format.
- `Memory Updated` and `Graph Updated` are booleans (`true` or `false`).
- `Files Changed` contains multiple file paths separated by commas. Literal commas within a file path must be escaped as `\,`. Literal backslashes must be escaped as `\\`. Escaping order: backslashes first, then commas. Unescaping order: commas first (`\,` → `,`), then backslashes (`\\` → `\`). Consumers **must** split on unescaped commas to obtain individual file paths.
- Literal pipe characters (`|`) in any field must be escaped as `\|`.

### Empty State

`No completed tasks yet.` is the canonical empty-state marker. It is non-record content (no leading `-` or pipe separators) and **must** be ignored by parsers rather than treated as a task entry.
