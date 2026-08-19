# Graph Context

This directory is the structural source of truth for the codebase, containing node/edge JSON maps.

**Rule:** The AI must NEVER manually edit these files. Graph updates are handled strictly and atomically by the [[runtime/graph-updater.md]] phase after an accepted change.
