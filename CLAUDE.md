# Project Brain v2

Project Brain is an AI-native engineering runtime that eliminates repeated codebase analysis by maintaining a continuously synchronized knowledge system.

## Project Structure
```
project-brain/
├── system/       # Core pipeline and orchestrator rules
├── graph/        # Structural source of truth (JSON graphs)
├── memory/       # Modular knowledge files
├── tasks/        # Task history and tracking
├── standards/    # Engineering quality rules
├── reviews/      # Review checklists and scoring
├── templates/    # Markdown templates for records
├── runtime/      # Execution engine components
└── cache/        # Optimization caches
```

Key knowledge files: [[memory/architecture|Architecture]], [[memory/frontend|Frontend]], [[memory/checkpoints|Checkpoints]]. Runtime engine components: [[runtime/graph-updater|Graph Updater]], [[runtime/memory-updater|Memory Updater]].

## Naming Conventions
- **Files and Directories**: `kebab-case` for utility files, config files, and directories. `PascalCase` for components/classes. `camelCase` for data/hooks.
- **Code**: Functions/methods use verbs (`getUser`). Booleans use prefixes like `is`, `has`, `should`. Constants use `UPPER_SNAKE_CASE`. Types/interfaces use `PascalCase`.
- **References**: Use markdown link syntax for internal file references (`[[file-name]]`).

## Global AI Tooling & Git Rules
- **DO NOT ANALYZE ENTIRE CODEBASE**: Always retrieve only the required context using graph retrieval and modular memory.
- **DETERMINISTIC PIPELINE**: Every prompt must follow the established runtime orchestrator pipeline (`runtime/orchestrator.md`). No stage may be skipped.
- **INCREMENTAL UPDATES**: Documentation updates automatically after accepted changes. Entire documentation is never regenerated.
- **STRUCTURAL INTEGRITY**: When adding new permanent architectural files or directories, you MUST immediately update `CLAUDE.md` and relevant directory documentation to reflect the new project structure.
- **ATTRIBUTION POLICY**: NEVER add `Co-Authored-By`, `Contributor`, or similar attribution referencing AI in commit messages or docs. All work should appear human-authored.
