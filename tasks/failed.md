# Failed Tasks

Records tasks that were rejected or abandoned.

## Format

```
- [Task ID] | Task Description | Reason for failure | Date (YYYY-MM-DD) | Lessons learned
```

**Example:**
```
- T-042 | Implement login flow | Auth provider deprecated mid-implementation | 2026-07-10 | Verify third-party provider stability before starting integration work\; keep alternatives researched upfront
```

`Date` must use `YYYY-MM-DD` format. `Lessons learned` is free-form text; multiple lessons within it are separated by semicolons. 

### Escaping Grammar

All pipe-delimited record fields follow the same escaping rules:

1. **Escaping order**: backslashes first, then pipes, then (for `Lessons learned`) semicolons.
2. **Literal backslash** → `\\`
3. **Literal pipe** → `\|`
4. **Literal semicolon** (within `Lessons learned` only) → `\;`

**Unescaping order** (reverse of escaping):

1. Split record fields on unescaped pipe delimiters.
2. For each field, unescape in order: `\|` → `|`, then `\\` → `\`.
3. For the `Lessons learned` field, additionally split on unescaped semicolons, then unescape `\;` → `;` and `\\` → `\` per segment.

Consumers **must** follow this order to preserve literal backslashes, pipes, and semicolons unambiguously.

### Empty State

`No failed tasks.` is the canonical empty-state marker. It is non-record content (no leading `-` or pipe separators) and **must** be ignored by parsers rather than treated as a task entry.
