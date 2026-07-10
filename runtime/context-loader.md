# Context Loader

Loads only the relevant memory files required for the current task.

## Node-to-Memory Mapping Policy

### Resolution Rules

Each `AffectedNode` from Graph Retriever resolves to a memory file as follows:

1. Strip the `type` prefix from `id` to produce a base name (e.g., `module:auth` → `auth`).
2. Append `.md` and resolve relative to `memory/` (e.g., `memory/auth.md`).
3. If no direct match, fall back to the `path` field — extract the filename stem (e.g., `src/routes/login.ts` → `login`) and look for `memory/login.md`.

### Directory Escape Enforcement

- All resolved paths are normalized and checked against `memory/` as the root.
- If a resolved path would escape `memory/` (e.g., `../../etc/passwd`), throw `EscapeError` and return no context.
- Symbolic links within `memory/` are followed, but the target must also reside under `memory/`.

### Missing and Duplicate Files

- **Missing file**: Silently skipped — no error raised.
- **Duplicate**: If two nodes resolve to the same file, load it once and include it once.

### Affected-Node-Only Loading

- Only nodes in `AffectedNode[]` are resolved. No additional memory files are loaded.
- Assembled context concatenates loaded file contents, each prefixed with its path as a header.

### Empty-Graph Fallback Mode

Fallback is entered only when `graphEmpty === true` (the explicit signal from Graph Retriever). It is **not** triggered by an empty `AffectedNode[]` from a non-empty graph.

1. Load **all** `memory/*.md` files as a fallback.
2. Apply the same path-safety enforcement as normal resolution: normalize each resolved path and verify it stays under `memory/`; throw `EscapeError` immediately and return no context otherwise.
3. Silent skip any `memory/*.md` that fails to load due to missing or unreadable file — I/O errors only; EscapeError and other non-I/O failures are never swallowed.
4. Fallback mode is only entered when `graphEmpty === true` — it is not a general "load everything" override.
