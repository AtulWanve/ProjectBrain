# Active Tasks

Tracks currently in-progress work.

## Format

```
- [Task ID] | Description | Started (YYYY-MM-DD HH:MM) | Priority (High/Medium/Low) | Status (In Progress/Blocked/Pending Review)
```

Literal backslash characters (`\`) must be escaped as `\\`. Literal pipe characters (`|`) in the `Description` field must be escaped as `\|`. Escaping order: backslashes first, then pipes. Unescaping order: pipes first, then backslashes. Parsers **must** split on unescaped pipe delimiters, then unescape `\|` to `|`, then unescape `\\` to `\`.

**Example:**
```
- T-042 | Implement login flow | 2026-07-10 14:30 | High | In Progress
```

### Empty State

When there are no active tasks, the file contains the literal text `No active tasks.` with no preceding `-` or pipe separators. This is non-record content and **must** be ignored by parsers rather than treated as a task entry.
