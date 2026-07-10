# Changelog

Records all significant changes to the project.

## Format

```
YYYY-MM-DD | Task ID | Description | Type (feat/fix/refactor/docs)
```

Literal backslash characters (`\`) must be escaped as `\\`. Literal pipe characters (`|`) in the `Description` field must be escaped as `\|`. Escaping order: backslashes first, then pipes. Unescaping order: pipes first, then backslashes. Consumers **must** split on unescaped pipe delimiters, then unescape `\|` to `|`, then unescape `\\` to `\`.

### Empty State

`No entries yet.` is the canonical empty-changelog marker. It is non-record content (no pipe separators) and **must** be ignored by parsers rather than treated as a changelog entry.
