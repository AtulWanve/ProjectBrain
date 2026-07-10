# Recent Files

Tracks recently modified files for context continuity.

## Format

```
- <filepath> | <timestamp> | <change summary>
```

Literal `|` characters in `filepath` or `change summary` must be escaped as `\|`. Literal backslash characters (`\`) must be escaped as `\\`. Escaping order: backslashes first, then pipes. Unescaping order: pipes first, then backslashes.

**Example:**
```
- src/data/README.md | 2026-07-10 14:30 | Add usage examples for parse\\\|validate pipeline
```
The above example encodes a literal backslash followed by a literal pipe, i.e., the change summary is `Add usage examples for parse\|validate pipeline`.
- Encoding trace: original `\|` → escape backslashes first: `\\|` → escape pipes: `\\\|`
- Unescaping trace: `\\\|` → unescape pipes first: `\\|` → unescape backslashes: `\|`

## Retention

Keep at most the 20 most recent entries. When adding a new entry, remove the oldest entry if the cache already contains 20 entries.
