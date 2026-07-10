# Static Validator (Phase 12)

Before consuming AI review tokens, deterministic tooling validates the implementation.

## Validation Commands

Each command is executed with a deterministic contract:

| Property | Specification |
|----------|--------------|
| Node/npm version | As specified in the project's `.nvmrc` or `engines` field in `package.json`; if neither exists, the system-default LTS version |
| Working directory | Project root (`.`) |
| Setup step | Before validation runs, a separate setup step executes `npm ci` (using the lockfile) and verifies that `node_modules` is present. This setup step runs outside the validator — the validator itself does not perform installation. If `npm ci` fails or `node_modules` is missing, the setup step reports the error and validation is not attempted. |
| Lockfile | Must be present (`package-lock.json` or equivalent); the setup step uses `npm ci` instead of `npm install` to guarantee reproducible installs |
| Dependency state | Dependencies are installed (`node_modules` present) by the setup step before validation begins; the validator itself runs no installation logic and rejects execution if `node_modules` is absent |
| Environment | `NODE_ENV=test` (or equivalent) is explicitly set. Essential system variables (e.g., `PATH`, `HOME`, `USER`, `TMPDIR`) are inherited from the runner environment and preserved. No other environment variables are set unless explicitly required by the project configuration (documented in the project's own config). The validator uses a sanitized baseline: an explicit allowlist of permitted variables that includes essential runner variables, project configuration variables, and standard toolchain/CI variables. Any variable not on the allowlist is removed before validation. |
| Command ordering | The commands run sequentially in the listed order; if any command fails, the remaining commands are skipped |
| Timeout | Each command gets a 120-second timeout; if exceeded, the command is killed and treated as a failure |
| Cancellation | The runner must support SIGTERM for cancellation, with a 5-second grace period before SIGKILL |
| Result | Each execution returns `{ command: string; exitCode: number; stdout: string; stderr: string }`; the full stdout and stderr are captured |

### Commands

- `npm run lint` — code quality and style
- `npm run typecheck` — TypeScript type safety
- `npm run build` — compilation success
- `npm run test` — test suite passes

## Behavior

- If any validation fails, execution returns to the Planning phase for correction (starting a new full pass through the pipeline from Planning forward)
- AI review is not entered on that pass when validation fails
- This minimizes token consumption by catching errors early
- Returning to an earlier stage does **not** violate the orchestrator's no-stage-skipping guarantee: every stage is visited in order on each full pass; a failed validation simply initiates a new pass through the pipeline from the re-entry point.

## Rules

- Validation is deterministic — same code always produces same result
- No subjective evaluation during validation
- Fail fast on the first validation error
- Report exact error messages for debugging
