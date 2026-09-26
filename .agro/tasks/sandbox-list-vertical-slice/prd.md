# PRD: Sandbox List Vertical Slice

Status: DRAFT

## User Stories

### US-001: Lock the sandbox list contract

**Description:** As a CLI maintainer, I want tests for the `sandbox list` contract so that I can move its code without changing its behavior.

**Acceptance Criteria:**

- [ ] Tests cover `sandbox list` with no entries, text rows, JSON rows, and a failed status probe.
- [ ] Tests cover the current `sandbox list` argument and help behavior, including `--json`, unexpected positionals, and unknown flags.
- [ ] Tests assert the exact JSON field names and order, text output, exit codes, and relevant stderr output.
- [ ] The new tests pass against the original implementation before any production code moves.

### US-002: Isolate sandbox list without changing its contract

**Description:** As a CLI maintainer, I want a bounded `sandbox list` command slice so that later commands can follow a tested pattern.

**Acceptance Criteria:**

- [ ] `cli.ts` retains top-level command selection but delegates `sandbox list` behavior to a focused module under `.agro/cli/src/commands/`.
- [ ] The slice owns list-specific dispatch and output. `runSandboxInstall` remains in its current command module.
- [ ] The slice reuses `listEntries`, `entryRoot`, configuration readers, and `ExecutionTarget`. It does not add a second registry or execution implementation.
- [ ] The old `sandbox list` behavior does not remain as a second implementation in `sandbox.ts` or `cli.ts`.
- [ ] Tests from US-001 pass without changed assertions after the move.
- [ ] `pnpm run typecheck`, `pnpm exec vitest run .agro/cli/src/__tests__/sandbox.test.ts`, and `pnpm run build:harness` exit with code 0.

### US-003: Give sandbox commands one controller

**Description:** As a CLI maintainer, I want a sandbox command controller so that sandbox routing stays outside `cli.ts`.

**Acceptance Criteria:**

- [ ] `.agro/cli/src/controllers/sandbox.ts` owns sandbox argument parsing, sandbox help, and dispatch to install or list.
- [ ] `cli.ts` selects `sandbox` and calls the controller without inspecting its subcommand or arguments.
- [ ] Existing exports of `parseSandboxArgs` and `printSandboxHelp` remain available to current callers.
- [ ] Sandbox install and list behavior, including error output, help output, and exit codes, stays unchanged.
- [ ] The full test suite, typecheck, and CLI build exit with code 0.

### US-004: Select an explicit sandbox version

**Description:** As an operator, I want to name a sandbox and a release version so that an upgrade targets one known image.

**Acceptance Criteria:**

- [ ] The sandbox parser accepts `upgrade <name> --version <X.Y.Z>` with a validated release version.
- [ ] The parser supports the same leading `v` and prerelease format as `sandbox install --version`.
- [ ] Missing or invalid versions, missing names, `--latest`, and extra arguments return non-zero with a diagnostic.
- [ ] Sandbox help names the upgrade command; `agro update` retains its CLI-only behavior.

### US-005: Apply the image and preserve state

**Description:** As an operator, I want the selected image applied immediately so that the sandbox runs it without losing home data.

**Acceptance Criteria:**

- [ ] The command refuses absent or build-mode entries without changing their config.
- [ ] A successful command starts the selected image through the existing Compose wrapper, then writes `image.ref` to the registry entry.
- [ ] The command preserves the home mount, named volumes, checkout, `.env`, and unrelated config fields; it never invokes `down -v`.
- [ ] Failed provisioning retains the prior `image.ref`, attempts restoration, and reports restoration failure when restoration also fails.
- [ ] Same-entry concurrent upgrades cannot apply conflicting versions; different entries can upgrade independently.
- [ ] A conflicting `AGRO_SANDBOX_IMAGE` from the host shell or entry `.env` cannot silently replace the selected version.

### US-006: Document and verify the upgrade

**Description:** As an operator, I want tests and instructions so that I know when the upgrade changes a running sandbox.

**Acceptance Criteria:**

- [ ] Tests cover parsing, success, failure, restoration, and persistent home data with an injected runner.
- [ ] `docs/lifecycle-commands.md` warns that container recreation interrupts active sessions and preserves home data.
- [ ] `.agro/cli/README.md` documents the command and distinguishes it from CLI self-upgrade.
- [ ] Typecheck, full tests, build, and applicable eval probes exit with code 0.

## Summary

The current `.agro/cli/src/cli.ts` parses `sandbox` arguments and dispatches `sandbox list`. The current `.agro/cli/src/commands/sandbox.ts` loads registry entries, reads their config, checks execution status, and renders text or JSON. Existing tests in `.agro/cli/src/__tests__/sandbox.test.ts` cover text and JSON rows. Extend those tests before moving production code. Keep the CLI contract unchanged. The operator requires a `controllers/` directory in the CLI. The sandbox controller owns sandbox parsing, help, and dispatch. The operator later approved an explicit, immediate sandbox image upgrade in this same PR. The new command uses the controller and a focused service without changing `agro update`.

## Key Integration Points

| File | Function(s) / Symbol(s) | Role |
|---|---|---|
| `.agro/cli/src/cli.ts` | `parseSandboxArgs`, `main` | Preserve parsing and top-level selection; remove list-specific dispatch from the entry point. |
| `.agro/cli/src/commands/sandbox.ts` | `runSandboxList`, `runSandboxInstall` | Move list behavior; keep install behavior in place. |
| `.agro/cli/src/commands/sandbox-list.ts` | `runSandboxList` | Own list execution and rendering after extraction. |
| `.agro/cli/src/controllers/sandbox.ts` | `parseSandboxArgs`, `printSandboxHelp`, `runSandboxCommand` | Own sandbox command parsing, help, and dispatch. |
| `.agro/cli/src/services/sandbox-upgrade.ts` | `runSandboxUpgrade` | Coordinate image selection, registry state, lifecycle application, and recovery. |
| `.agro/cli/src/lib/version.ts`, `.agro/cli/src/lib/agro-config.ts` | `parseReleaseVersion`, `officialImageRef`, `ImageSettings` | Reuse validated image and config types. |
| `.agro/cli/src/lib/registry.ts` | `listEntries`, `entryRoot`, `registryRoot` | Keep registry reads in the existing source of truth. |
| `.agro/cli/src/lib/execution/target.ts` | `ExecutionTarget.status` | Keep status probes behind the existing execution boundary. |
| `.agro/cli/src/__tests__/sandbox.test.ts` | `agro sandbox list` | Lock behavior before extraction. |

## Interface Integration Points

| Surface | Change Type | Description |
|---|---|---|
| `agro sandbox list [--json]` | Internal only | Preserve arguments, help, output, and exit codes. |
| `.agro/cli/src/commands/sandbox-list.ts` | New internal module | Establish a focused list operation. |
| `.agro/cli/src/controllers/sandbox.ts` | New internal module | Own sandbox command parsing, help, and dispatch. |
| `agro sandbox upgrade <name> --version <X.Y.Z>` | New CLI command | Recreate one image-mode sandbox at the explicit release version. |

## Storage

The sandbox registry remains under `${AGRO_HOME:-~/.agro}/sandboxes/`. The upgrade changes the existing `image.ref` field only after a successful image application. `registry.ts` remains the registry access point. A per-entry lock prevents concurrent upgrades of one sandbox.

## Architectural Decisions

- Keep the command workflow as the unit of organization. Do not add HTTP-style routes, controllers, or models to the CLI.
- Keep one parser contract. If extraction moves `parseSandboxArgs`, preserve its install behavior and exported API; do not create a second parser for list.
- Keep `cli.ts` responsible for selecting the top-level command. The sandbox controller owns sandbox parsing, help, and dispatch. Keep list formatting and status lookup in the focused list module.
- Preserve current behavior even where the parser accepts flags that `sandbox list` does not use.
- For upgrade, accept an explicit sandbox name and version only; refuse build-mode entries. Stage the candidate image before persisting `image.ref`.
- Use the current Compose wrapper and `ExecutionTarget`. On failure, keep the prior config and attempt to restore the previous image.
- Use `ImageSettings` in `lib/agro-config.ts` as the model and `lib/version.ts` for pure version functions. Put upgrade coordination in `services/sandbox-upgrade.ts`; do not add empty model or utility directories.
- Run code and tests inside the sandbox. This task does not start a service or change host lifecycle behavior.

## Test Plan (TDD)

| Test File | Case(s) | Validates |
|---|---|---|
| `.agro/cli/src/__tests__/sandbox.test.ts` | Empty registry, aligned text rows, JSON row keys and order, status probe failure | List rendering and status behavior. |
| `.agro/cli/src/__tests__/cli.property.test.ts` | List argument parsing for `--json`, extra arguments, and unknown flags | Existing parser contract. |
| `.agro/cli/src/__tests__/cli-first-help.test.ts` | `sandbox` help and list dispatch output | Public command behavior after extraction. |
| `.agro/cli/src/__tests__/lifecycle.test.ts` | Sandbox install and list argument contracts; upgrade flags | Existing and new controller behavior. |
| `.agro/cli/src/__tests__/sandbox.test.ts` | Success, failure, home preservation, and concurrent upgrade cases | Upgrade state and lifecycle behavior. |

Add tests that detect output or dispatch changes; run the tests against the original code first. Then move the list code and rerun the same tests. Run `pnpm run typecheck` and `pnpm run build:harness` from the repository root.

## Design Principles

- Change one command group, not the full CLI. Keep the CLI upgrade and sandbox image upgrade distinct.
- Reuse the registry and execution boundaries.
- Delete moved code instead of keeping duplicate implementations.
- Add no explanatory comments to tracked code.
- Keep host and sandbox boundaries unchanged. The command works after a terminal disconnect and does not share mutable state between agents.

## Out of Scope

- Refactoring `sandbox install` behavior or other command families.
- Adding application scaffolds, routes, controllers, models, services, or repositories.
- Changing existing CLI flags, list output, or registry layout. Add only the approved upgrade command and matching documentation.
- Adding dependencies or changing the packaged CLI build layout.

## Open Questions

None.

## Acceptance Criteria

- [ ] US-001 tests pass against the original implementation before the code move.
- [ ] All six stories meet every listed criterion.
- [ ] An explicit sandbox image upgrade applies immediately without changing home storage or the CLI executable.
- [ ] `pnpm run typecheck` exits with code 0 from the repository root.
- [ ] `pnpm exec vitest run .agro/cli/src/__tests__/sandbox.test.ts .agro/cli/src/__tests__/cli.property.test.ts .agro/cli/src/__tests__/cli-first-help.test.ts` exits with code 0 from the repository root.
- [ ] `pnpm run build:harness` exits with code 0 from the repository root.
- [ ] `git diff` shows no change to public CLI output contracts or persisted data.

## Lessons

- Claim: A probe that scans a source path can fail when the implementation moves without changing behavior. Evidence: `agro-sandbox-image-mode.sh` failed in CI after sandbox parsing moved from `cli.ts` to `controllers/sandbox.ts`; the updated probe passed against the controller and failed against missing flags. Outcome: fixed in this PR.
